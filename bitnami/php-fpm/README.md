# PHP-FPM packaged by Bitnami

## What is PHP-FPM?

> PHP-FPM (FastCGI Process Manager) is an alternative PHP FastCGI implementation with some additional features useful for sites of any size, especially busier sites.

[Overview of PHP-FPM](http://php-fpm.org/)

Trademarks: This software listing is packaged by Bitnami. The respective trademarks mentioned in the offering are owned by the respective companies, and use of them does not imply any affiliation or endorsement.

## TL;DR

```console
$ docker run -it --name phpfpm -v /path/to/app:/app bitnami/php-fpm
```

### Docker Compose

```console
$ curl -sSL https://raw.githubusercontent.com/bitnami/containers/main/bitnami/php-fpm/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

## Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading Linux distribution.
* All Bitnami images available in Docker Hub are signed with [Docker Content Trust (DCT)](https://docs.docker.com/engine/security/trust/content_trust/). You can use `DOCKER_CONTENT_TRUST=1` to verify the integrity of the images.
* Bitnami container images are released on a regular basis with the latest distribution packages available.

## Supported tags and respective `Dockerfile` links

Learn more about the Bitnami tagging policy and the difference between rolling tags and immutable tags [in our documentation page](https://docs.bitnami.com/tutorials/understand-rolling-tags-containers/).


* [`8.1`, `8.1-debian-11`, `8.1.11`, `8.1.11-debian-11-r0`, `latest` (8.1/debian-11/Dockerfile)](https://github.com/bitnami/containers/blob/main/bitnami/php-fpm/8.1/debian-11/Dockerfile)
* [`8.0`, `8.0-debian-11`, `8.0.23`, `8.0.23-debian-11-r11` (8.0/debian-11/Dockerfile)](https://github.com/bitnami/containers/blob/main/bitnami/php-fpm/8.0/debian-11/Dockerfile)
* [`7.4`, `7.4-debian-11`, `7.4.30`, `7.4.30-debian-11-r46` (7.4/debian-11/Dockerfile)](https://github.com/bitnami/containers/blob/main/bitnami/php-fpm/7.4/debian-11/Dockerfile)

Subscribe to project updates by watching the [bitnami/containers GitHub repo](https://github.com/bitnami/containers).

### Deprecation Note (2022-01-21)

The `prod` tags has been removed; from now on just the regular container images will be released.

### Deprecation Note (2020-08-18)

The formatting convention for `prod` tags has been changed:

* `BRANCH-debian-10-prod` is now tagged as `BRANCH-prod-debian-10`
* `VERSION-debian-10-rX-prod` is now tagged as `VERSION-prod-debian-10-rX`
* `latest-prod` is now deprecated

## Get this image

The recommended way to get the Bitnami PHP-FPM Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/bitnami/php-fpm).

```console
$ docker pull bitnami/php-fpm:latest
```

To use a specific version, you can pull a versioned tag. You can view the [list of available versions](https://hub.docker.com/r/bitnami/php-fpm/tags/) in the Docker Hub Registry.

```console
$ docker pull bitnami/php-fpm:[TAG]
```

If you wish, you can also build the image yourself by cloning the repository, changing to the directory containing the Dockerfile and executing the `docker build` command. Remember to replace the `APP`, `VERSION` and `OPERATING-SYSTEM` path placeholders in the example command below with the correct values.

```console
$ git clone https://github.com/bitnami/containers.git
$ cd bitnami/APP/VERSION/OPERATING-SYSTEM
$ docker build -t bitnami/APP:latest .
```

## Connecting to other containers

This image is designed to be used with a web server to serve your PHP app, you can use docker networking to create a network and attach all the containers to that network.

### Serving your PHP app through an nginx frontend

We will use PHP-FPM with nginx to serve our PHP app. Doing so will allow us to setup more complex configuration, serve static assets using nginx, load balance to different PHP-FPM instances, etc.

#### Step 1: Create a network

```console
$ docker network create app-tier --driver bridge
```

or using Docker Compose:

```yaml
version: '2'

networks:
  app-tier:
    driver: bridge
```

#### Step 2: Create a server block

Let's create an nginx server block to reverse proxy to our PHP-FPM container.

```nginx
server {
  listen 0.0.0.0:80;
  server_name myapp.com;

  root /app;

  location / {
    try_files $uri $uri/index.php;
  }

  location ~ \.php$ {
    # fastcgi_pass [PHP_FPM_LINK_NAME]:9000;
    fastcgi_pass phpfpm:9000;
    fastcgi_index index.php;
    include fastcgi.conf;
  }
}
```

Notice we've substituted the link alias name `myapp`, we will use the same name when creating the container.

Copy the server block above, saving the file somewhere on your host. We will mount it as a volume in our nginx container.

#### Step 3: Run the PHP-FPM image with a specific name

Docker's linking system uses container ids or names to reference containers. We can explicitly specify a name for our PHP-FPM server to make it easier to connect to other containers.

```console
$ docker run -it --name phpfpm \
  --network app-tier
  -v /path/to/app:/app \
  bitnami/php-fpm
```

or using Docker Compose:

```yaml
services:
  phpfpm:
    image: 'bitnami/php-fpm:latest'
    networks:
      - app-tier
    volumes:
      - /path/to/app:/app
```

#### Step 4: Run the nginx image

```console
$ docker run -it \
  -v /path/to/server_block.conf:/opt/bitnami/nginx/conf/server_blocks/yourapp.conf \
  --network app-tier \
  bitnami/nginx
```

or using Docker Compose:

```yaml
services:
  nginx:
    image: 'bitnami/nginx:latest'
    depends_on:
      - phpfpm
    networks:
      - app-tier
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - /path/to/server_block.conf:/opt/bitnami/nginx/conf/server_blocks/yourapp.conf
```

## PHP runtime

Since this image bundles a PHP runtime, you may want to make use of PHP outside of PHP-FPM. By default, running this image will start a server. To use the PHP runtime instead, we can override the the default command Docker runs by stating a different command to run after the image name.

### Entering the REPL

PHP provides a REPL where you can interactively test and try things out in PHP.

```console
$ docker run -it --name phpfpm bitnami/php-fpm php -a
```

**Further Reading:**

- [PHP Interactive Shell Documentation](http://php.net/manual/en/features.commandline.interactive.php)

## Running your PHP script

The default work directory for the PHP-FPM image is `/app`. You can mount a folder from your host here that includes your PHP script, and run it normally using the `php` command.

```console
$ docker run -it --name php-fpm -v /path/to/app:/app bitnami/php-fpm \
  php script.php
```

## Configuration

### Mount a custom config file

You can mount a custom config file from your host to edit the default configuration for the php-fpm docker image. The following is an example to alter the configuration of the _php-fpm.conf_ configuration file:

#### Step 1: Run the PHP-FPM image

Run the PHP-FPM image, mounting a file from your host.

```console
$ docker run --name phpfpm -v /path/to/php-fpm.conf:/opt/bitnami/php/etc/php-fpm.conf bitnami/php-fpm
```

or by modifying the [`docker-compose.yml`](https://github.com/bitnami/containers/blob/main/bitnami/php-fpm/docker-compose.yml) file present in this repository:

```yaml
services:
  phpfpm:
  ...
    volumes:
      - /path/to/php-fpm.conf:/opt/bitnami/php/etc/php-fpm.conf
  ...
```

#### Step 2: Edit the configuration

Edit the configuration on your host using your favorite editor.

```console
$ vi /path/to/php-fpm.conf
```

#### Step 3: Restart PHP-FPM

After changing the configuration, restart your PHP-FPM container for the changes to take effect.

```console
$ docker restart phpfpm
```

or using Docker Compose:

```console
$ docker-compose restart phpfpm
```

### Add additional .ini files

PHP has been configured at compile time to scan the `/opt/bitnami/php/etc/conf.d/` folder for extra .ini configuration files so it is also possible to mount your customizations there.

Multiple files are loaded in alphabetical order. It is common to have a file per extension and use a numeric prefix to guarantee an order loading the configuration.

Please check [http://php.net/manual/en/configuration.file.php#configuration.file.scan](http://php.net/manual/en/configuration.file.php#configuration.file.scan) to know more about this feature.

In order to override the default `max_file_uploads` settings you can do the following:

1. Create a file called _custom.ini_ with the following content:

```config
max_file_uploads = 30M
```

2. Run the php-fpm container mounting the custom file.

```console
$ docker run -it -v /path/to/custom.ini:/opt/bitnami/php/etc/conf.d/custom.ini bitnami/php-fpm php -i | grep max_file_uploads

```

You should see that PHP is using the new specified value for the `max_file_uploads` setting.

## Logging

The Bitnami PHP-FPM Docker Image sends the container logs to the `stdout`. You can configure the containers [logging driver](https://docs.docker.com/engine/reference/run/#logging-drivers-log-driver) using the `--log-driver` option. By defauly the `json-file` driver is used.

To view the logs:

```console
$ docker logs phpfpm
```

or using Docker Compose:

```console
$ docker-compose logs phpfpm
```

*The `docker logs` command is only available when the `json-file` or `journald` logging driver is in use.*

## Maintenance

### Upgrade this image

Bitnami provides up-to-date versions of PHP-FPM, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container.

#### Step 1: Get the updated image

```console
$ docker pull bitnami/php-fpm:latest
```

or if you're using Docker Compose, update the value of the image property to
`bitnami/php-fpm:latest`.

#### Step 2: Stop and backup the currently running container

Stop the currently running container using the command

```console
$ docker stop php-fpm
```

or using Docker Compose:

```console
$ docker-compose stop php-fpm
```

Next, take a snapshot of the persistent volume `/path/to/php-fpm-persistence` using:

```console
$ rsync -a /path/to/php-fpm-persistence /path/to/php-fpm-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

You can use this snapshot to restore the database state should the upgrade fail.

#### Step 3: Remove the currently running container

```console
$ docker rm -v phpfpm
```

or using Docker Compose:

```console
$ docker-compose rm -v phpfpm
```

#### Step 4: Run the new image

Re-create your container from the new image.

```console
$ docker run --name phpfpm bitnami/php-fpm:latest
```

or using Docker Compose:

```console
$ docker-compose up phpfpm
```

## Useful Links

- [Create An AMP Development Environment With Bitnami Containers
](https://docs.bitnami.com/containers/how-to/create-amp-environment-containers/)
- [Create An EMP Development Environment With Bitnami Containers
](https://docs.bitnami.com/containers/how-to/create-emp-environment-containers/)

## Notable Changes

### 7.2.3-r2, 7.1.15-r2, 7.0.28-r2 and 5.6.34-r2 (2018-03-13)

- PHP has been configured at compile time to scan the `/opt/bitnami/php/etc/conf.d/` folder for extra .ini configuration files.

### 7.0.6-r0 (2016-05-17)

- All volumes have been merged at `/bitnami/php-fpm`. Now you only need to mount a single volume at `/bitnami/php-fpm` for persistence.
- The logs are always sent to the `stdout` and are no longer collected in the volume.

### 5.5.30-2 (2015-12-07)

- Enables support for imagick extension

### 5.5.30-0-r01 (2015-11-10)

- `php.ini` is now exposed in the volume mounted at `/bitnami/php-fpm/conf/` allowing users to change the defaults as per their requirements.

### 5.5.30-0 (2015-10-06)

- `/app` directory is no longer exported as a volume. This caused problems when building on top of the image, since changes in the volume are not persisted between Dockerfile `RUN` instructions. To keep the previous behavior (so that you can mount the volume in another container), create the container with the `-v /app` option.

## Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/containers/issues), or submit a [pull request](https://github.com/bitnami/containers/pulls) with your contribution.

## Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/containers/issues/new/choose). For us to provide better support, be sure to fill the issue template.

## License

Copyright &copy; 2022 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
