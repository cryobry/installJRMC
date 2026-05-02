# installJRMC

Installs [JRiver Media Center](https://www.jriver.com/) and associated services on most Linux distributions.

You can find the latest version of installJRMC, changelog, and documentation in [my repository](https://git.bryanroessler.com/bryan/installJRMC).

## tl;dr

`curl https://git.bryanroessler.com/bryan/installJRMC/raw/branch/master/installJRMC|bash`

## Usage

```text
$ installJRMC --help
USAGE:
  installJRMC [[OPTION] [VALUE]]...

  installJRMC defaults to --install=repo on platforms with a JRiver repository and --install=local on others.
  Specifying --build, --createrepo, --service, or --uninstall disables the default install method.

OPTIONS
  --install, -i repo|local
    repo: Install MC from repository, updates are handled by the system package manager.
    local: Build and install MC locally from official source package.
  --build[=suse|fedora|centos|mandriva]
    Build RPM from source DEB but do not install.
    Optionally, specify a target distro for cross-building (ex. --build=suse, note the '=').
  --compat
    Build/install MC locally without minimum dependency version requirements.
  --mcversion VERSION
    Specify the MC version, ex. "35.0.74" or "35" (default: latest release).
  --arch VERSION
    Specify the target MC architecture, ex. "amd64", "arm64", etc (default: host).
  --mcrepo REPO
    Specify the MC repository, ex. "bullseye", "bookworm", "noble", etc (default: auto).
  --outputdir PATH
    Generate reusable installJRMC output in this PATH (default: ./output).
  --restorefile MJR_FILE
    Restore file location for automatic license registration.
  --betapass PASSWORD
    Enter beta team password for access to beta builds.
  --service, -s SERVICE
    See SERVICES below for possible services to install.
  --service-type user|system
    Starts services at boot (system) or at user login (user) (default: per service, see SERVICES).
  --no-update
    Disable automatic installJRMC self-update.
  --uninstall, -u
    Uninstall JRiver MC, remove services, containers, and firewall rules (does not remove library files).
  --yes, -y, --auto
    Assume yes response to questions.
  --version, -v
    Print installJRMC version and exit.
  --debug, -d
    Print debug output.
  --help, -h
    Print help dialog and exit.

SERVICES
  jriver-mediaserver (default --service-type=user)
    Enable and start a mediaserver systemd service (requires an existing X server).
  jriver-mediacenter (user)
    Enable and start a mediacenter systemd service (requires an existing X server).
  jriver-x11vnc (user)
    Enable and start x11vnc for the local desktop (requires an existing X server).
    Usually combined with jriver-mediaserver or jriver-mediacenter services.
    --vncpass and --display are optional (see below).
  jriver-xvnc (system)
    Enable and start a new Xvnc session running JRiver Media Center.
    --vncpass PASSWORD
      Set the vnc password for x11vnc/Xvnc access. If no password is set, installJRMC
      will either use existing password stored in $HOME/.vnc/jrmc_passwd or else no password.
    --display DISPLAY
      Display to use for x11vnc/Xvnc (default: The current display (x11vnc) or the
      current display incremented by 1 (Xvnc)).
  jriver-createrepo (system)
    Install hourly service to build latest MC RPM and run createrepo.

ADVANCED OPTIONS
  --container, -c CONTAINER (TODO: Under construction)
    See CONTAINERS section below for a list of possible services to install.
  --createrepo[=suse|fedora|centos|mandriva]
    Build rpm, copy to webroot, and run createrepo. 
    Use in conjunction with --build=TARGET for crossbuilding repos.
    Optionally, specify a target distro for non-native repo (ex. --createrepo=fedora).
  --createrepo-webroot PATH
    Specify the webroot directory to install the repo (default: /var/www/jriver).
  --webroot-user USER
    Owner/user for createrepo output in the webroot (default: current user).
  --createrepo-user USER
    Backward-compatible alias for --webroot-user.
  --sign
    Sign the built RPM and repodata/repomd.xml (if --createrepo).
  --sign-user USER
    User account used to run rpmsign and gpg signing (default: current user).
  --sign-key KEYID
    GPG key ID, fingerprint, or UID used for --sign.
```

## Notes

### --service-type=

Services use a sane default `--service-type` listed next to the service name in the [`--service=`](#--service) section. User services begin at user login and are managed by the unprivileged user, for example: `systemctl --user stop jriver-mediacenter`. System services begin at boot and are managed by root, for example: `sudo systemctl stop jriver-servicename@username.service`. It is possible to run all services of a particular user at boot using [`sudo loginctl enable-linger username`](https://www.freedesktop.org/software/systemd/man/loginctl.html).

Multiple services (but not `--service-types`) can be installed using multiple `--service` blocks: `installJRMC --install=repo --service=jriver-x11vnc --service=jriver-mediacenter`

### jriver-x11vnc versus jriver-xvnc

[jriver-x11vnc](http://www.karlrunge.com/x11vnc/) shares the existing X display via VNC and can be combined with additional services to start Media Center or Media Server. Conversely, [jriver-xvnc](https://tigervnc.org/doc/Xvnc.html) creates a new Xvnc display and starts a JRiver Media Center service in the foreground of the new VNC display.

### Firewall

`installJRMC` automatically creates port forwarding firewall rules for remote access to Media Network (52100-52200/tcp, 1900/udp DLNA) and Xvnc/x11vnc (if selected), using `firewall-cmd` or `ufw` (if available).

### Other Nicities

* Automatically updates `installJRMC` to the latest release.
* Activates external third-party repositories for improved media playback (hardware decoding, etc.).
* Adds temporary legacy repositories to provide deprecated libraries.
* Symlinks non-standard SSL certs.
* Activates MC if a valid license file is found in common locations like Downloads or Documents directories.

### Containers

**Coming soon!**

## Examples

* `installJRMC`

    Install the latest version of MC from the best available repository.

* `installJRMC --mcversion 33 --debug`

    Install the latest version of MC33 from the best available repository with debugging output.

* `installJRMC --install local --compat`

    Install a more widely-compatible version of the latest MC (for older distros).

* `installJRMC --install repo --service jriver-mediacenter --service-type user`

    Install MC from the repository and start/enable `jriver-mediacenter.service` as a user service.

* `installJRMC --install local --compat --restorefile /path/to/license.mjr --mcversion 35.0.74`

    Build and install an MC 35.0.74 compatibility RPM locally and activate it using the `/path/to/license.mjr`.

* `installJRMC --createrepo --createrepo-webroot /srv/jriver/repo --webroot-user www-user`

    Build an RPM locally for the current distro, move it to the webroot, and run createrepo as `www-user`.

* `installJRMC --service=jriver-createrepo --webroot-user=nginx --createrepo-webroot "/var/www/repos.bryanroessler.com/jriver" --createrepo=fedora --sign --sign-key 7C008BD7B33C185837C81D033C9D3657EE0BC6D9`

    Install the jriver-createrepo timer and service to build/sign the RPM, move it to the webroot, and run createrepo as user `nginx` hourly.

* `installJRMC --createrepo --webroot-user nginx --sign --sign-user bryan --sign-key 0xDEADBEEF`

    Build/update the RPM repo, sign both RPM and repodata as `bryan`, and publish files owned by `nginx`.

* `installJRMC --install repo --service jriver-x11vnc --service jriver-mediacenter --vncpass "letmein"`

    Install services to share the existing local desktop via VNC and automatically run MC on startup.

* `installJRMC --install repo --service jriver-xvnc --display ":2"`

    Install an Xvnc server on display ':2' that starts MC.

* `installJRMC --uninstall`

    Uninstall MC, services, and firewall rules. This will **not** remove your media, media library/database, or library backup folder.

## Additional Info

Find a bug? [Let me know on Interact!](https://yabb.jriver.com/interact/index.php/topic,141168.0.html)

Find `installJRMC` useful? [Paypal me a coffee!](https://paypal.me/bryanroessler)

[↓ ↓ ↓ Bitcoin ↓ ↓ ↓](bitcoin:bc1q7wy0kszjavgcrvkxdg7mf3s6rh506rasnhfa4a)

[![Bitcoin](https://repos.bryanroessler.com/files/bc1q7wy0kszjavgcrvkxdg7mf3s6rh506rasnhfa4a.png)](bitcoin:bc1q7wy0kszjavgcrvkxdg7mf3s6rh506rasnhfa4a)
