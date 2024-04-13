# 1.0.0 #

## Added ##

### Debian ###

- Added the `debian-security.sources` sources file to the `/etc/apt/sources.list.d/` directory.
- Added the group called `administrator`.
- Added the user called `ilyabakhlinlebedev` belonging to the `administrator` group.

### Dependencies Build ###

- Added the 6.4 version of the [`ncurses`](https://github.com/mirror/ncurses) library.
- Added the 8.2 version of the [`readline`](https://git.savannah.gnu.org/cgit/readline.git/) library.

### Dependencies Installation ###

- Added the 8.2 version of the [`readline`](https://git.savannah.gnu.org/cgit/readline.git/) library.

### Docker ###

- Added the default `-a` entrypoint command.

### PHP ###

- Added the build and installation of the PHP CLI.
- Added the default development INI file to the `/usr/local/etc/` directory.

## Changed ##

### Debian ###

- Changed the sources `debian.sources` file at the `/etc/apt/sources.list.d/` directory.
- Changed the default execution user from `root:root` to `ilyabakhlinlebedev:administrator`.
- Changed the default work directory from `/` to `/home/ilyabakhlinlebedev/`.

### Docker ###

- Changed the default entrypoint from `/bin/bash` to `/usr/local/bin/php`.
