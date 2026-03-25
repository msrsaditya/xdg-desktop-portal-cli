project('xdg-desktop-portal-cli', 'c', version: '1.0.0', default_options: ['c_std=c11'])
add_project_arguments(
  '-D_POSIX_C_SOURCE=200809L',
  '-DHAVE_LIBSYSTEMD=1',
  '-DSYSCONFDIR="/etc"',
  language: 'c'
)
cc = meson.get_compiler('c')
libsystemd = dependency('libsystemd')
inih = dependency('inih')
prefix = get_option('prefix')
libexecdir = join_paths(prefix, get_option('libexecdir'))
datadir = join_paths(prefix, get_option('datadir'))
systemduserunitdir = join_paths(prefix, 'lib', 'systemd', 'user')
dbusservicedir = join_paths(datadir, 'dbus-1', 'services')
portaldir = join_paths(datadir, 'xdg-desktop-portal', 'portals')
conf_data = configuration_data()
conf_data.set('libexecdir', libexecdir)
inc = include_directories('include')
executable('xdg-desktop-portal-cli',
  ['src/core/config.c', 'src/core/logger.c', 'src/core/main.c', 'src/core/request.c', 'src/filechooser/filechooser.c'],
  include_directories: inc,
  dependencies: [libsystemd, inih],
  install: true,
  install_dir: libexecdir
)
configure_file(input: 'xdg-desktop-portal-cli.service.in', output: 'xdg-desktop-portal-cli.service', configuration: conf_data, install_dir: systemduserunitdir)
configure_file(input: 'org.freedesktop.impl.portal.desktop.cli.service.in', output: 'org.freedesktop.impl.portal.desktop.cli.service', configuration: conf_data, install_dir: dbusservicedir)
install_data('cli.portal', install_dir: portaldir)
