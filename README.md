# installJRMC

This program will install [JRiver Media Center](https://www.jriver.com/) (JRMC) and associated services on most major Linux distros.

## Executing

`installJRMC [--option [ARGUMENT]]`

Running `installJRMC` without any options will install the latest version of JRiver Media Center from the official JRiver repository (Ubuntu/Debian) or my [unofficial repository](https://repos.bryanroessler.com/jriver/) (Fedora/CentOS) using the system package manager (`--install repo`). If any other option is specified, then the default install method (i.e. `--install repo` or `--install local`) will need to be explicitly specified. This makes it possible to install services and containers independent of MC.

**Note**: As of v1.0b14 major version library migrations are performed if the destination config directory `$HOME/.jriver/Media Center XX` is missing for major release `XX`. However, it is still a good idea to create a manual library backup before migrating major versions.

## Options

```text
$ installJRMC --help
--install, -i repo|local
    repo: Install MC from repository, future updates will be handled by the system package manager
    local: Build and install MC package locally from official source package
--build[=suse|fedora|centos]
    Build RPM from source DEB but do not install
    Optionally, specify a target distro for cross-building (ex. --build=suse, note the '=')
--compat
    Build/install MC without minimum dependency version requirements
--mcversion VERSION
    Build or install a specific MC version, ex. "30.0.55" (default: latest version)
--outputdir PATH
    Generate rpmbuild output in this PATH (default: ./output)
--restorefile RESTOREFILE
    Restore file location for automatic license registration
--betapass PASSWORD
    Enter beta team password for access to beta builds
--service, -s SERVICE
    See SERVICES section below for the list of services to deploy
  --service-type user|system
      Starts services at boot (system) or user login (user) (default: per-service, see SERVICES)
--container, -c CONTAINER (TODO: Under construction)
    See CONTAINERS section below for a list of containers to deploy
--createrepo[=suse|fedora|centos]
    Build rpm, copy to webroot, and run createrepo.
    Optionally, specify a target distro for non-native repo (ex. --createrepo=fedora, note the '=')
  --createrepo-webroot PATH
      The webroot directory to install the repo (default: /var/www/jriver/)
  --createrepo-user USER
      The web server user if different from the current user
--version, -v
    Print this script version and exit
--debug, -d
    Print debug output
--help, -h
    Print help dialog and exit
--uninstall, -u
    Uninstall JRiver MC, cleanup service files, and remove firewall rules (does not remove library or media files)
```

## Services

```text
jriver-mediaserver (default --service-type=user)
    Enable and start a mediaserver systemd service (requires an existing X server)
jriver-mediacenter (user)
    Enable and start a mediacenter systemd service (requires an existing X server)
jriver-x11vnc (user)
    Enable and start x11vnc for the local desktop (requires an existing X server, does NOT support Wayland)
  --vncpass and --display are also valid options (see below)
jriver-xvnc (system)
    Enable and start a new Xvnc session running JRiver Media Center
  --vncpass PASSWORD
    Set vnc password for x11vnc/Xvnc access. If no password is set, the script will either use existing password stored in ~/.vnc/jrmc_passwd or use no password
  --display DISPLAY
    Manually specify display to use for x11vnc/Xvnc (ex. ':1')
jriver-createrepo (system)
    Install hourly service to build latest MC RPM and run createrepo
    By default installs as root service to handle www permissions more gracefully
```

### `--service-type`

MC helper services are installed as system-level services (`--service-type system`) by default and are manipulable as admin: `sudo systemctl stop jriver-servicename@username.service`. It is also possible to create user-level services using `--service-type user` that can be manipulated by the unprivileged user: `systemctl --user stop jriver-mediacenter`.

Multiple services (but not `--service-types`) can be installed at one time using multiple `--service` blocks: `installJRMC --install repo --service jriver-x11vnc --service jriver-mediacenter`

### `jriver-x11vnc` versus `jriver-xvnc`

[jriver-x11vnc](http://www.karlrunge.com/x11vnc/) shares your existing X display via VNC and can be combined with additional services to start Media Center or Media Server. Conversely, [jriver-xvnc](https://tigervnc.org/doc/Xvnc.html) creates a new Xvnc display and starts a JRiver Media Center service in the foreground of the new VNC display.

**Note**: If `jriver-xvnc` finds an existing display it will attempt to increment the display number by 1. This should work fine in most cases, but if you have multiple running X servers on your host machine you should use the `--display` option to specify a free display.

## Containers

**Coming soon!**

## Firewall

`installJRMC` will automatically add port forwarding firewall rules enabling remote access to Media Server (52100-52200/tcp, 1900/udp DLNA) and Xvnc/x11vnc (depends on port selection). `installJRMC` uses `firewall-cmd` on EL distros and `ufw` on Debian/Ubuntu.

**Note:** `ufw` is not installed by default on Debian but will be installed by `installJRMC`. To prevent user lock-out (i.e. SSH), Debian users that have not already enabled `ufw` will need to `sudo ufw enable` after running `installJRMC` and inspecting their configuration.

## Examples

* `installJRMC`

    Install the latest version of MC from the best available repository.

* `installJRMC --install local --compat`

    Install a more widely-compatible version of the latest MC (for older distros).

* `installJRMC --install repo --service jriver-mediacenter --service-type user`

    Install MC from the repository and start/enable `jriver-mediacenter.service` as a user service.

* `installJRMC --install local --compat --restorefile /path/to/license.mjr --mcversion 30.0.55`

    Build and install an MC 30.0.55 comptability RPM locally and activate it using the `/path/to/license.mjr`

* `installJRMC --createrepo --createrepo-webroot /srv/jriver/repo --createrepo-user www-user`

     Build an RPM locally for the current distro, move it to the webroot, and run createrepo as `www-user`.

* `installJRMC --service jriver-createrepo --createrepo-webroot /srv/jriver/repo --createrepo-user www-user`

    Install the jriver-createrepo timer and service to build the RPM, move it to the webroot, and run createrepo as `www-user` hourly.

* `installJRMC --install repo --service jriver-x11vnc --service jriver-mediacenter --vncpass "letmein"`

    Install services to share the existing local desktop via VNC and automatically run MC on startup.

* `installJRMC --install repo --service jriver-xvnc --display ":2"`

    Install an Xvnc server on display ':2' that starts MC.

* `installJRMC --uninstall`

    Uninstall MC and its associated services and firewall rules. This will **not** remove your media, media library/database, or automated library backup folder.

## Additional Info

Did you find `installJRMC` useful? [Buy me a coffee!](https://paypal.me/bryanroessler?locale.x=en_US)

Did you find a bug? Let me know on [Interact!](https://yabb.jriver.com/interact/index.php/topic,134152.0.html)
