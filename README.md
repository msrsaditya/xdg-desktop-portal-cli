# xdg-desktop-portal-cli

A custom XDG Desktop Portal backend that allows GUI applications to use a terminal-based file manager (like `lf`, `ranger`, or `nnn`) as their native file chooser. 

It is designed primarily for keyboard-driven environments and explicitly targets the Sway window manager, but you can easily make little modifications to the source code to make it work to any desktop environment of your choice. When an application requests to open or save a file via the standard D-Bus portal interface, this backend intercepts the request and launches a specified shell script/terminal UI for file selection.

## 🚀 Features
* Integrates terminal file managers into the standard Freedesktop portal ecosystem.
* Supports both `OpenFile` and `SaveFile` portal requests.
* Lightweight, written in C11.
* Configurable via a simple INI file.

## ⚠️ Development Status
**Note:** This project is currently an experimental prototype. There are known architectural areas being actively improved:
* **Security:** Currently relies on `system()` for executing the file-picker script, which may be vulnerable to shell injection if applications pass malformed paths.
* **Concurrency:** Uses a hardcoded temporary file (`/tmp/cli.portal`) for inter-process communication, which can cause race conditions if multiple file choosers are opened simultaneously.
* **Event Loop:** The D-Bus event loop blocks while the terminal file picker is open.

*Use in a trusted, single-user environment until these issues are resolved.*

## 📦 Prerequisites

To build and run this project, you will need the following dependencies installed on your system:

* A C compiler supporting **C11**
* **Meson** & **Ninja** (Build system)
* **libsystemd** (For D-Bus communication)
* **inih** (For parsing the configuration file)

On Fedora/RHEL, you can install these with:
```bash
sudo dnf install gcc meson ninja-build systemd-devel inih-devel
```

## 🛠️ Building and Installation

This project uses Meson for its build system. 

1. Clone the repository:
   ```bash
   git clone https://github.com/YourUsername/xdg-desktop-portal-cli.git
   cd xdg-desktop-portal-cli
   ```

2. Setup the build directory:
   ```bash
   meson setup builddir --prefix=/usr
   ```

3. Compile the project:
   ```bash
   ninja -C builddir
   ```

4. Install the binaries, D-Bus service files, and systemd units (requires root):
   ```bash
   sudo ninja -C builddir install
   ```

## ⚙️ Configuration

The portal backend looks for a configuration file in your XDG config directory, typically located at `~/.config/xdg-desktop-portal-cli/config`.

Create the directory and file:
```bash
mkdir -p ~/.config/xdg-desktop-portal-cli
touch ~/.config/xdg-desktop-portal-cli/config
```

Add the following to the `config` file, modifying the paths to match your system:

```ini
[filechooser]
# The script or command to launch your terminal file manager
cmd=/home/username/.local/bin/lf/file-picker.sh

# The default directory to open when saving a file if none is provided
default_dir=/tmp
```

### The File-Picker Script
Your `cmd` script will be executed with the following arguments:
1. `multiple`: `1` if multiple files can be selected, `0` otherwise.
2. `directory`: `1` if the user is selecting a directory, `0` otherwise.
3. `writing`: `1` if this is a "Save File" dialog, `0` for "Open File".
4. `path`: The requested starting path (or current file name).
5. `portal_path`: The hardcoded path (`/tmp/cli.portal`) where your script **must** write the selected absolute file paths (one per line).

## 🖥️ Usage

By default, the `cli.portal` file declares `UseIn=sway`.

To ensure applications use this portal, make sure your environment variables are set correctly in your compositor startup script or `.profile`:

```bash
export XDG_CURRENT_DESKTOP=sway
```

When an application attempts to open a file dialog, D-Bus will automatically start the `xdg-desktop-portal-cli` service, which will parse your config and launch your terminal file picker.

## 📜 Logging & Troubleshooting

If things aren't working, you can manually run the executable with a higher log level to see what is happening:

```bash
/usr/libexec/xdg-desktop-portal-cli --loglevel=TRACE
```
