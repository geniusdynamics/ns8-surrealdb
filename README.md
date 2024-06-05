# ns8-surrealdb

This is a template module for [NethServer 8](https://github.com/NethServer/ns8-core).

## Install

Instantiate the module with:

```shell
add-module ghcr.io/geniusdynamics/surrealdb:latest 1
```

The output of the command will return the instance name.
Output example:

    {"module_id": "surrealdb", "image_name": "surrealdbngx", "image_url": "ghcr.io/geniusdynamics/surrealdbngx:latest"}

## Configure

Let's assume that the mattermost instance is named `Surrealdb`.

Launch `configure-module`, by setting the following parameters:
- `host`: a fully qualified domain name for the application
- `http2https`: enable or disable HTTP to HTTPS redirection (true/false)
- `lets_encrypt`: enable or disable Let's Encrypt certificate (true/false)
- `SURREALDB_NAME`: this is the surrealdb username
- `SURREALDB_PASSWORD`: this is the surrealdb password for authentication


Example:

```shell
api-cli run configure-module --agent module/surrealdb1 --data - <<EOF
{
  "host": "surreal.domain.com",
  "http2https": true,
  "lets_encrypt": false,
  "SURREALDB_NAME": "root,
  "SURREALDB_PASSWORD": root,
}
EOF
```

The above command will:
- start and configure the surrealdb instance
- configure a virtual host for trafik to access the instance

## Get the configuration
You can retrieve the configuration with

```shell
api-cli run get-configuration --agent module/surrealdb1
```

## Update

```shell
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/surrealdb:latest","instances":["surrealdb1"],"force":true}'
```

## Uninstall

To uninstall the instance:
```shell
remove-module --no-preserve surrealdb1
```
    

## Smarthost setting discovery

Some configuration settings, like the smarthost setup, are not part of the
`configure-module` action input: they are discovered by looking at some
Redis keys.  To ensure the module is always up-to-date with the
centralized [smarthost
setup](https://geniusdynamics.github.io/ns8-core/core/smarthost/) every time
surrealdb starts, the command `bin/discover-smarthost` runs and refreshes
the `state/smarthost.env` file with fresh values from Redis.

Furthermore if smarthost setup is changed when surrealdb is already
running, the event handler `events/smarthost-changed/10reload_services`
restarts the main module service.

See also the `systemd/user/surrealdb.service` file.

This setting discovery is just an example to understand how the module is
expected to work: it can be rewritten or discarded completely.

## Debug

some CLI are needed to debug

- The module runs under an agent that initiate a lot of environment variables (in /home/surrealdb1/.config/state), it could be nice to verify them
on the root terminal

    `runagent -m surrealdb1 env`

- you can become runagent for testing scripts and initiate all environment variables
  
    `runagent -m surrealdb1`

 the path become : 
```
    echo $PATH
    /home/surrealdb1/.config/bin:/usr/local/agent/pyenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/
```

- if you want to debug a container or see environment inside
 `runagent -m surrealdb1`
 ```
podman ps
CONTAINER ID  IMAGE                                      COMMAND               CREATED        STATUS        PORTS                    NAMES
d292c6ff28e9  localhost/podman-pause:4.6.1-1702418000                          9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  80b8de25945f-infra
d8df02bf6f4a  docker.io/library/mariadb:10.11.5          --character-set-s...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  mariadb-app
9e58e5bd676f  docker.io/library/nginx:stable-alpine3.17  nginx -g daemon o...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  surrealdb-app
```

you can see what environment variable is inside the container
```
podman exec  surrealdb-app env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
PKG_RELEASE=1
MARIADB_DB_HOST=127.0.0.1
MARIADB_DB_NAME=surrealdb
MARIADB_IMAGE=docker.io/mariadb:10.11.5
MARIADB_DB_TYPE=mysql
container=podman
NGINX_VERSION=1.24.0
NJS_VERSION=0.7.12
MARIADB_DB_USER=surrealdb
MARIADB_DB_PASSWORD=surrealdb
MARIADB_DB_PORT=3306
HOME=/root
```

you can run a shell inside the container

```
podman exec -ti   surrealdb-app sh
/ # 
```
## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/geniusdynamics/surrealdb:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project
