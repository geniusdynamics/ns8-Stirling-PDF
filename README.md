# ns8-stirlingpdf



## Install

Instantiate the module with:

    add-module ghcr.io/compgeniuses/stirlingpdf:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "Stilingpdf1", "image_name": "Stilingpdf", "image_url": "ghcr.io/geniusdynamics/Stilingpdf:latest"}

## Configure

Let's assume that the mattermost instance is named `stirlingpdf1`.

Launch `configure-module`, by setting the following parameters:
- `host`: a fully qualified domain name for the application
- `http2https`: enable or disable HTTP to HTTPS redirection (true/false)
- `lets_encrypt`: enable or disable Let's Encrypt certificate (true/false)
- `docker_enable_security`: Define if the app will have a password (true/false)
- `security_enablelogin`: Define if the app will have a password (true/false)
- `security_initiallogin_username`:  define the initial username , if enable login is defined
- `security_initiallogin_password`: define initial password

## Default COnfig if enable login is enabled
USername: admin
PAssword: stirling

Example:

```
api-cli run configure-module --agent module/stirlingpdf1 --data - <<EOF
{
  "host": "stirlingpdf.domain.com",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

The above command will:
- start and configure the stirlingpdf instance
- configure a virtual host for trafik to access the instance

## Get the configuration
You can retrieve the configuration with

```
api-cli run get-configuration --agent module/stirlingpdf1
```

## Update Module
You can retrieve the configuration with

```
api-cli run update-module --data '{"module_url":"ghcr.io/compgeniuses/stirlingpdf:latest","instances":["stirlingpdf1"],"force":true}'
```

## Uninstall

To uninstall the instance:

    remove-module --no-preserve stirlingpdf1

## Smarthost setting discovery

Some configuration settings, like the smarthost setup, are not part of the
`configure-module` action input: they are discovered by looking at some
Redis keys.  To ensure the module is always up-to-date with the
centralized [smarthost
setup](https://compgeniuses.github.io/ns8-core/core/smarthost/) every time
stirlingpdf starts, the command `bin/discover-smarthost` runs and refreshes
the `state/smarthost.env` file with fresh values from Redis.

Furthermore if smarthost setup is changed when stirlingpdf is already
running, the event handler `events/smarthost-changed/10reload_services`
restarts the main module service.

See also the `systemd/user/stirlingpdf.service` file.

This setting discovery is just an example to understand how the module is
expected to work: it can be rewritten or discarded completely.

## Debug

some CLI are needed to debug

- The module runs under an agent that initiate a lot of environment variables (in /home/stirlingpdf1/.config/state), it could be nice to verify them
on the root terminal

    `runagent -m stirlingpdf1 env`

- you can become runagent for testing scripts and initiate all environment variables
  
    `runagent -m stirlingpdf1`

 the path become : 
```
    echo $PATH
    /home/stirlingpdf1/.config/bin:/usr/local/agent/pyenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/
```

- if you want to debug a container or see environment inside
 `runagent -m stirlingpdf1`
 ```
podman ps
CONTAINER ID  IMAGE                                      COMMAND               CREATED        STATUS        PORTS                    NAMES
d292c6ff28e9  localhost/podman-pause:4.6.1-1702418000                          9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  80b8de25945f-infra
d8df02bf6f4a  docker.io/library/mariadb:10.11.5          --character-set-s...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  mariadb-app
9e58e5bd676f  docker.io/library/nginx:stable-alpine3.17  nginx -g daemon o...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  stirlingpdf-app
```

you can see what environment variable is inside the container
```
podman exec  stirlingpdf-app env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
PKG_RELEASE=1
MARIADB_DB_HOST=127.0.0.1
MARIADB_DB_NAME=stirlingpdf
MARIADB_IMAGE=docker.io/mariadb:10.11.5
MARIADB_DB_TYPE=mysql
container=podman
NGINX_VERSION=1.24.0
NJS_VERSION=0.7.12
MARIADB_DB_USER=stirlingpdf
MARIADB_DB_PASSWORD=stirlingpdf
MARIADB_DB_PORT=3306
HOME=/root
```

you can run a shell inside the container

```
podman exec -ti   stirlingpdf-app sh
/ # 
```
## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/compgeniuses/stirlingpdf:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a compgeniuses developer to add it to ns8 Weblate project
