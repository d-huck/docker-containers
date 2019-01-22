# Sonarr
![Alt text](https://cdn.rawgit.com/Sonarr/Sonarr/develop/Logo/256.png "")

- [Quick Start](#quick-start)
- [Description](#description)
- [What is in this image?](#what-is-in-this-image)
- [Supported Architectures](#supported-architectures)
- [Contributing](#contributing)
- [Issues](#issues)
- [Installation](#installation)
    - [Install as current user](#install-as-current-user)
    - [Install as other user](#install-as-other-user)
    - [Install from GitHub](#install-from-github)
    - [Initial Configuration](#initial-configuration)
    - [Adding more Volumes](#adding-more-volumes)
- [Maintenance](#maintenance)
    - [Upgrading](#upgrading)
    - [Automatic Upgrades](#automatic-upgrades)
    - [Removal](#removal)
    - [Shell Access](#shell-access)
    - [Logs](#logs)
    - [List Running processes](#list-running-processes)
- [Technical Information](#technical-information)
    - [Environment Variables](#environment-variables)
        - [Adjusting Variables](#adjusting-variables)
    - [Volumes](#volumes)
    - [Manual Run](#manual-run)
- [License](#license)
- [Donation](#donation)

## Quick Start:
![demo](img/preview/sonarr-demo.gif)
```
docker run -v /usr/local/bin:/target --entrypoint instl hurricane/sonarr
sonarr
```

## Description:

Sonarr is a PVR for Usenet and BitTorrent users. It can monitor multiple RSS
feeds for new episodes of your favorite shows and will grab, sort and rename
them. It can also be configured to automatically upgrade the quality of files
already downloaded when a better quality format becomes available.

This subfolder contains all necessary files to build
a [Docker](https://www.docker.com/) image for
[sonarr](https://github.com/sonarr/sonarr).

## What is in this image:

This image is built from the official sonarr release and includes all
prescribed prerequisites.

## Supported Architectures:

amd64, x86_64, arm32v7, arm64v8

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D

## Issues

Before reporting your issue please try updating Docker to the latest version
and check if it resolves the issue. Refer to the Docker [installation
guide](https://docs.docker.com/installation) for instructions.

SELinux users should try disabling SELinux using the command `setenforce 0` to
see if it resolves the issue.

If the above recommendations do not help then [report your
issue](../../issues/new) along with the following information:

- Output of the `docker version` and `docker info` commands
- The `docker run` command or `docker-compose.yml` used to start the image.
  Mask out the sensitive bits.
- Please state if you are using [Boot2Docker](http://www.boot2docker.io),
  [VirtualBox](https://www.virtualbox.org), etc.

## Installation:
For the best experience install directly from [Docker
Hub](https://hub.docker.com/r/hurricane/nzbget/).

The installation process and scripts are very versatile and can be adjusted by
passing the right combination of variables and arguments to each of the
commands.

The following examples should cover most scenarios. In each scenario, a wrapper
script will be installed on the host that should ease creation and management
of the containerized application. When executing the script it will create
a container named `sonarr`. Additionally, the script will ensure that this
container gets setup with the appropriate environment variables and volumes
each time it is executed.

#### Installation as current user:
Start the installation by issuing the following command from within a terminal:
```sh
docker run -it --rm -v /usr/local/bin:/target \
    --entrypoint instl
    hurricane/sonarr
```

Optionally, you can install a systemd service file by executing:
```sh
docker run -it --rm -v /etc/systemd/system:/target  \
    --entrypoint instl
    hurricane/sonarr service
```

To enable the systemd service for `sonarr` execute the following:
```sh
sudo systemctl enable sonarr@$USER
```

#### Install as another user:
In the following instructions adjust each command replacing `username` with the
name of the user you wish the container to run as.

To install the application execute and, again, adjust the command replacing
`username` accordingly.
```sh
docker run -it --rm -v /usr/local/bin:/target \
    -e "APP_USER=username" \
    --entrypoint instl
    hurricane/sonarr
```

Note, if the user is a system account, the command will need further
adjustment. This is because by default the script stores settings and
configuration in a hidden folder within the executing user's home directory.
This can be overridden by passing the appropriate environment variable
(`APP_CONFIG`) to the `instl` script, such as in the example below:
```sh
docker run -it --rm -v /usr/local/bin:/target \
    -e "APP_USER=username" \
    -e "APP_CONFIG=/var/lib/nzbget" \
    --entrypoint instl
    hurricane/sonarr
```

### [Install from GitHub](https://github.com/hurricane/docker-containers/sonarr):
Installation from GitHub is recommended only for the purposes of
troubleshooting and development. To install from GitHub execute the
following:
```sh
git clone https://github.com/hurricane/docker-containers
cd docker-containers
./build.sh sonarr
```

Then proceed by following any of the docker run commands described in the
installation instructions above.

### Initial Configuration:
Once the wrapper script for the sonarr docker has been installed you just need
to execute the wrapper script from within a terminal:
```sh
sonarr
```
On the first run, the script will prompt for system paths that you wish made
accessible from within the container. Enter one path per line.

#### Adding more volumes:
Volumes which should be mounted within the container at runtime are kept in the
volume configuration file found under the `APP_CONFIG` folder on the host. The
location will vary depending on the type of installation.

If the wrapper script was installed as the executing user the volume
configuration file can be found at:
`${HOME}/.config/sonarr/.sonarr.volumes`
Otherwise at:
`${APP_CONFIG}/.sonarr.volumes`

## Maintenance

### Upgrading:
You can upgrade the version of sonarr found within the container by executing
one of the following commands:
```sh
sonarr update
```

Or by executing:
```sh
docker exec sonarr update
```

You can update the container itself by executing:
```sh
docker pull hurricane/sonarr
docker stop sonarr
sonarr
```

### Automatic Upgrades:
If you wish the docker container to automatically update upon creation, set the
environment variable `EDGE` to `1`. Please read the `Technical Details` section
for the various ways this can be achieved.

In order to have the container periodically check and upgrade sonarr one needs
to add a [`crontab`](https://en.wikipedia.org/wiki/Cron) entry. Like so:
```sh
echo "0 2 * * * docker exec sonarr update" | sudo tee -a /var/spool/cron/crontabs/root
```
or
```sh
echo "0 2 * * * sonarr update" | sudo tee -a /var/spool/cron/crontabs/root
```
### Removal:
```sh
docker run -it --rm \
  --volume /usr/local/bin:/target \
  --entrypoint uninstl
  hurricane/sonarr
```

### Shell Access
For debugging and maintenance purposes you may want to access the container's
shell. If you are using Docker version `1.3.0` or higher you can access
a running container's shell by starting `bash` using `docker exec`:

```sh
sonarr --console
```
or
```sh
docker exec -it sonarr bash
```

### Logs
```sh
sonarr --logs
```

### List running processes:
```sh
sonarr --status
```

## Technical information:

By default the containerized application has been set to run with UID and GID
`1000`. If using the automatic install method from Docker, the container is set
to run with the UID and GID of of the user executing the `sonarr` wrapper
script.  Additionally, the wrapper script saves sonarr's configuration and
settings in a hidden sub folder in the executing user's home directory. Most
default settings can be adjusted by passing the appropriate environment
variable. Here is a list of any and all applicable environment variables that
can be override by the end user.

### Environment Variables:
You can adjust some of the default settings set for container/application by
passing any or all of the following environment variable:

| ENV VAR        | Definition                                                                     |
| -------------- | ------------------------------------------------------------------------------ |
| APP_USER       | Name of user the service will run as.\[4\]                                     |
| APP_UID        | UID assigned to APP_USER upon creation, or will query APP_USER's ID.\[3\]      |
| APP_GID        | GID assigned to APP_USER upon creation, or will query APP_USER's GID.\[3\]     |
| APP_CONFIG     | Location where application will store settings and database on host.\[1\]      |
| APP_GUEST_CFG  | Location where application will store settings and database within guest.\[4\] |
| APP_PORT       | App's Web UI port used to configure and access the service.\[2\]               |
| APP_SSL_PORT   | App's Web UI SSL port used to configure and access the service.\[2\]           |
| UMASK          | umask assigned to service, default set to 002.\[4\]                            |
| EDGE           | Update the containerized service, default set to 0(Off).\[4\]                  |

\[1\]: Variable is applicable only during install.  
\[2\]: Variable is applicable during install, when invoking installed wrapper script or systemd service.  
\[3\]: Variable is applicable only when invoking docker run directly.  
\[4\]: Variable is applicable in all scenarios.  

#### Adjusting Variables:
If you are unfamiliar with passing environment variables when invoking `docker
run` please refer to Docker's documentation on [environment
variables](https://docs.docker.com/engine/reference/run/#env-environment-variables)
for clarification. Otherwise, the following examples should be pretty clear.

The following examples will use the environment variable `EDGE`. `EDGE` has
been chosen since it is applicable during all scenarios.

To pass the `EDGE` variable when invoking `docker run` append the following
prior to the image name. Any and all other applicable variables can be done in
the same manner.
```sh
--env=EDGE=1
```

To pass the environment variable during other scenarios do so like in one of
the examples below:

From the commandline when calling the wrapper script:
```
EDGE=1 sonarr
```

By adjusting the systemd service:
```ini
[Service]
Type=simple
Environment=EDGE=1
...
```

### Volumes:
* `/config`  - Folder for configuration and settings.


### Manual Run:
Of course you can always run the docker image manually. Please be aware that if
you wish your data to remain persistent you need to provide a location for the
`/config` volume. For example,
```
docker run -d --net=host \
    -v /*your_config_location*:/config \
    -e TZ=America/Edmonton
    --name=sonarr hurricane/sonarr
```
All the information mention previously regarding user UID and GID still applies
when executing a docker run command.


# License

Code released under the [MIT license](./LICENSE).


# Donation
[@hurricanehrndz](https://github.com/hurricanehrndz): [![PayPal](https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=74S5RK533DD6C)
