# WordPress with NGINX packaged by Bitnami

## What is WordPress with NGINX?

> WordPress with NGINX combines the most popular blogging application with the power of the NGINX web server.

[Overview of WordPress with NGINX](http://www.wordpress.org)



## TL;DR

```console
$ curl -LO https://raw.githubusercontent.com/bitnami/bitnami-docker-wordpress-nginx/master/docker-compose.yml
$ docker-compose up
```

**Warning**: This quick setup is only intended for development environments. You are encouraged to change the insecure default credentials and check out the available configuration options in the [Environment Variables](#environment-variables) section for a more secure deployment.

## Why use Bitnami Images?

- Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
- With Bitnami images the latest bug fixes and features are available as soon as possible.
- Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
- All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading Linux distribution.
- All Bitnami images available in Docker Hub are signed with [Docker Content Trust (DCT)](https://docs.docker.com/engine/security/trust/content_trust/). You can use `DOCKER_CONTENT_TRUST=1` to verify the integrity of the images.
- Bitnami container images are released daily with the latest distribution packages available.

## Why use a non-root container?

Non-root container images add an extra layer of security and are generally recommended for production environments. However, because they run as a non-root user, privileged tasks are typically off-limits. Learn more about non-root containers [in our docs](https://docs.bitnami.com/tutorials/work-with-non-root-containers/).

## Supported tags and respective `Dockerfile` links

Learn more about the Bitnami tagging policy and the difference between rolling tags and immutable tags [in our documentation page](https://docs.bitnami.com/tutorials/understand-rolling-tags-containers/).


- [`5`, `5-debian-10`, `5.9.3`, `5.9.3-debian-10-r50`, `latest` (5/debian-10/Dockerfile)](https://github.com/bitnami/bitnami-docker-wordpress-nginx/blob/5.9.3-debian-10-r50/5/debian-10/Dockerfile)

Subscribe to project updates by watching the [bitnami/wordpress-nginx GitHub repo](https://github.com/bitnami/bitnami-docker-wordpress-nginx).

## Get this image

The recommended way to get the Bitnami WordPress with NGINX Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/bitnami/wordpress-nginx).

```console
$ docker pull bitnami/wordpress-nginx:latest
```

To use a specific version, you can pull a versioned tag. You can view the [list of available versions](https://hub.docker.com/r/bitnami/wordpress-nginx/tags/) in the Docker Hub Registry.

```console
$ docker pull bitnami/wordpress-nginx:[TAG]
```

If you wish, you can also build the image yourself.

```console
$ docker build -t bitnami/wordpress-nginx:latest 'https://github.com/bitnami/bitnami-docker-wordpress-nginx.git#master:5/debian-10'
```

## How to use this image

WordPress requires access to a MySQL or MariaDB database to store information. We'll use the [Bitnami Docker Image for MariaDB](https://www.github.com/bitnami/bitnami-docker-mariadb) for the database requirements.

### Run the application using Docker Compose

The main folder of this repository contains a functional [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-wordpress/blob/master/docker-compose.yml) file. Run the application using it as shown below:

```console
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-wordpress/master/docker-compose.yml > docker-compose.yml
$ docker-compose up
```

### Using the Docker Command Line

If you want to run the application manually instead of using `docker-compose`, these are the basic steps you need to run:

#### Step 1: Create a network

```console
$ docker network create wordpress-network
```

#### Step 2: Create a volume for MariaDB persistence and create a MariaDB container

```console
$ docker volume create --name mariadb_data
$ docker run -d --name mariadb \
  --env ALLOW_EMPTY_PASSWORD=yes \
  --env MARIADB_USER=bn_wordpress \
  --env MARIADB_PASSWORD=bitnami \
  --env MARIADB_DATABASE=bitnami_wordpress \
  --network wordpress-network \
  --volume mariadb_data:/bitnami/mariadb \
  bitnami/mariadb:latest
```

#### Step 3: Create volumes for WordPress persistence and launch the container

```console
$ docker volume create --name wordpress_data
$ docker run -d --name wordpress \
  -p 8080:8080 -p 8443:8443 \
  --env ALLOW_EMPTY_PASSWORD=yes \
  --env WORDPRESS_DATABASE_USER=bn_wordpress \
  --env WORDPRESS_DATABASE_PASSWORD=bitnami \
  --env WORDPRESS_DATABASE_NAME=bitnami_wordpress \
  --network wordpress-network \
  --volume wordpress_data:/bitnami/wordpress \
  bitnami/wordpress-nginx:latest
```

Access your application at `http://your-ip/`

## Persisting your application

If you remove the container all your data will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a directory at the `/bitnami/wordpress` path. If the mounted directory is empty, it will be initialized on the first run. Additionally you should [mount a volume for persistence of the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database).

The above examples define the Docker volumes named `mariadb_data` and `wordpress_data`. The WordPress application state will persist as long as volumes are not removed.

To avoid inadvertent removal of volumes, you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-wordpress-nginx/blob/master/docker-compose.yml) file present in this repository:

```diff
   mariadb:
     ...
     volumes:
-      - 'mariadb_data:/bitnami/mariadb'
+      - /path/to/mariadb-persistence:/bitnami/mariadb
   ...
   wordpress:
      ...
     volumes:
-      - 'wordpress_data:/bitnami/wordpress'
+      - /path/to/wordpress-persistence:/bitnami/wordpress
   ...
-volumes:
-  mariadb_data:
-    driver: local
-  wordpress_data:
-    driver: local
```

> NOTE: As this is a non-root container, the mounted files and directories must have the proper permissions for the UID `1001`.

### Mount host directories as data volumes using the Docker command line

#### Step 1: Create a network (if it does not exist)

```console
$ docker network create wordpress-network
```

#### Step 2. Create a MariaDB container with host volume

```console
$ docker run -d --name mariadb \
  --env ALLOW_EMPTY_PASSWORD=yes \
  --env MARIADB_USER=bn_wordpress \
  --env MARIADB_PASSWORD=bitnami \
  --env MARIADB_DATABASE=bitnami_wordpress \
  --network wordpress-network \
  --volume /path/to/mariadb-persistence:/bitnami/mariadb \
  bitnami/mariadb:latest
```

> NOTE: As this is a non-root container, the mounted files and directories must have the proper permissions for the UID `1001`.

#### Step 3. Create the WordPress container with host volumes

```console
$ docker run -d --name wordpress \
  -p 8080:8080 -p 8443:8443 \
  --env ALLOW_EMPTY_PASSWORD=yes \
  --env WORDPRESS_DATABASE_USER=bn_wordpress \
  --env WORDPRESS_DATABASE_PASSWORD=bitnami \
  --env WORDPRESS_DATABASE_NAME=bitnami_wordpress \
  --network wordpress-network \
  --volume /path/to/wordpress-persistence:/bitnami/wordpress \
  bitnami/wordpress-nginx:latest
```

> NOTE: As this is a non-root container, the mounted files and directories must have the proper permissions for the UID `1001`.

## Configuration

### Environment variables

When you start the WordPress image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the `docker run` command line. If you want to add a new environment variable:

- For docker-compose add the variable name and value under the application section in the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-wordpress/blob/master/docker-compose.yml) file present in this repository:

    ```yaml
    wordpress:
      ...
      environment:
        - WORDPRESS_PASSWORD=my_password
      ...
    ```

- For manual execution add a `--env` option with each variable and value:

    ```console
    $ docker run -d --name wordpress -p 80:8080 -p 443:8443 \
      --env WORDPRESS_PASSWORD=my_password \
      --network wordpress-tier \
      --volume /path/to/wordpress-persistence:/bitnami \
      bitnami/wordpress-nginx:latest
    ```

Available environment variables:

##### User and Site configuration

- `NGINX_HTTP_PORT_NUMBER`: Port used by NGINX for HTTP. Default: **8080**
- `NGINX_HTTPS_PORT_NUMBER`: Port used by NGINX for HTTPS. Default: **8443**
- `NGINX_ENABLE_ABSOLUTE_REDIRECT`: Use absolute URLs in Location header in redirections. Default: **no**
- `NGINX_ENABLE_PORT_IN_REDIRECT`: Use listening port in redirections issued by NGINX. Default: **no**
- `WORDPRESS_USERNAME`: WordPress application username. Default: **user**
- `WORDPRESS_PASSWORD`: WordPress application password. Default: **bitnami**
- `WORDPRESS_EMAIL`: WordPress application email. Default: **user@example.com**
- `WORDPRESS_FIRST_NAME`: WordPress user first name. Default: **FirstName**
- `WORDPRESS_LAST_NAME`: WordPress user last name. Default: **LastName**
- `WORDPRESS_BLOG_NAME`: WordPress blog name. Default: **User's blog**
- `WORDPRESS_DATA_TO_PERSIST`: Space separated list of files and directories to persist. Use a space to persist no data: `" "`. Default: **"wp-config.php wp-content"**
- `WORDPRESS_RESET_DATA_PERMISSIONS`: Force resetting ownership/permissions on persisted data when restarting WordPress, otherwise it assumes the ownership/permissions are correct. Ignored when running WP as non-root. Default: **no**
- `WORDPRESS_TABLE_PREFIX`: Table prefix to use in WordPress. Default: **wp_**
- `WORDPRESS_PLUGINS`: List of WordPress plugins to install and activate, separated via commas. Can also be set to `all` to activate all currently installed plugins, or `none` to skip. Default: **none**
- `WORDPRESS_EXTRA_INSTALL_ARGS`: Extra flags to append to the WordPress 'wp core install' command call. No defaults.
- `WORDPRESS_EXTRA_CLI_ARGS`: Extra flags to append to all WP-CLI command calls. No defaults.
- `WORDPRESS_EXTRA_WP_CONFIG_CONTENT`: Extra configuration to append to wp-config.php during install. No defaults.
- `WORDPRESS_ENABLE_HTTPS`: Whether to use HTTPS by default. Default: **no**
- `WORDPRESS_SKIP_BOOTSTRAP`: Skip the WordPress installation wizard. This is necessary when providing a database with existing WordPress data. Default: **no**
- `WORDPRESS_AUTO_UPDATE_LEVEL`: Level of auto-updates to allow for the WordPress core installation. Valid values: `major`, `minor`, `none`. Default: **none**

##### Multisite configuration

- `WORDPRESS_ENABLE_MULTISITE`: Enable WordPress Multisite configuration. Default: **no**
- `WORDPRESS_MULTISITE_HOST`: WordPress hostname/address. Only used for Multisite installations. No defaults.
- `WORDPRESS_MULTISITE_EXTERNAL_HTTP_PORT_NUMBER`: Port to used by WordPress to generate URLs and links when accessing using HTTP. Will be ignored if multisite mode is not enabled. Default **80**
- `WORDPRESS_MULTISITE_EXTERNAL_HTTPS_PORT_NUMBER`: Port to used by WordPress to generate URLs and links when accessing using HTTPS. Will be ignored if multisite mode is not enabled. Default **443**
- `WORDPRESS_MULTISITE_NETWORK_TYPE`: WordPress Multisite network type to enable. Valid values: `subfolder`, `subdirectory`, `subdomain`. Default: **subdomain**
- `WORDPRESS_MULTISITE_ENABLE_NIP_IO_REDIRECTION`: Whether to enable IP address redirection to nip.io wildcard DNS when enabling WordPress Multisite. This is useful when running on an IP address with subdomain network type. Default: **no**
- `WORDPRESS_MULTISITE_FILEUPLOAD_MAXK`: Maximum upload file size allowed for WordPress Multisite uploads, in kilobytes. Default: **81920**

##### Database connection configuration

- `WORDPRESS_DATABASE_HOST`: Hostname for the MariaDB or MySQL server. Default: **mariadb**
- `WORDPRESS_DATABASE_PORT_NUMBER`: Port used by the MariaDB or MySQL server. Default: **3306**
- `WORDPRESS_DATABASE_NAME`: Database name that WordPress will use to connect with the database. Default: **bitnami_wordpress**
- `WORDPRESS_DATABASE_USER`: Database user that WordPress will use to connect with the database. Default: **bn_wordpress**
- `WORDPRESS_DATABASE_PASSWORD`: Database password that WordPress will use to connect with the database. No defaults.
- `WORDPRESS_ENABLE_DATABASE_SSL`: Whether to enable SSL for database connections. Default: **no**
- `WORDPRESS_VERIFY_DATABASE_SSL`: Whether to verify the database SSL certificate when SSL is enabled for database connections. Default: **yes**
- `WORDPRESS_DATABASE_SSL_CERT_FILE`: Path to the database client certificate file. No defaults
- `WORDPRESS_DATABASE_SSL_KEY_FILE`: Path to the database client certificate key file. No defaults
- `WORDPRESS_DATABASE_SSL_CA_FILE`: Path to the database server CA bundle file. No defaults
- `ALLOW_EMPTY_PASSWORD`: It can be used to allow blank passwords. Default: **no**

##### Create a database for WordPress using mysql-client

- `MYSQL_CLIENT_FLAVOR`: SQL database flavor. Valid values: `mariadb` or `mysql`. Default: **mariadb**
- `MYSQL_CLIENT_DATABASE_HOST`: Hostname for the MariaDB or MySQL server. Default: **mariadb**
- `MYSQL_CLIENT_DATABASE_PORT_NUMBER`: Port used by the MariaDB or MySQL server. Default: **3306**
- `MYSQL_CLIENT_DATABASE_ROOT_USER`: Database admin user. Default: **root**
- `MYSQL_CLIENT_DATABASE_ROOT_PASSWORD`: Database password for the database admin user. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_NAME`: New database to be created by the mysql client module. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_USER`: New database user to be created by the mysql client module. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_PASSWORD`: Database password for the `MYSQL_CLIENT_CREATE_DATABASE_USER` user. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_CHARACTER_SET`: Character set to use for the new database. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_COLLATE`: Database collation to use for the new database. No defaults.
- `MYSQL_CLIENT_ENABLE_SSL_WRAPPER`: Whether to force SSL connections to the database via the `mysql` CLI tool. Useful for applications that rely on the CLI instead of APIs. Default: **no**
- `MYSQL_CLIENT_ENABLE_SSL`: Whether to force SSL connections for the database. Default: **no**
- `MYSQL_CLIENT_SSL_CA_FILE`: Path to the SSL CA file for the new database. No defaults
- `MYSQL_CLIENT_SSL_CERT_FILE`: Path to the SSL CA file for the new database. No defaults
- `MYSQL_CLIENT_SSL_KEY_FILE`: Path to the SSL CA file for the new database. No defaults
- `ALLOW_EMPTY_PASSWORD`: It can be used to allow blank passwords. Default: **no**

##### SMTP Configuration

To configure WordPress to send email using SMTP you can set the following environment variables:

- `WORDPRESS_SMTP_HOST`: SMTP host.
- `WORDPRESS_SMTP_PORT`: SMTP port.
- `WORDPRESS_SMTP_USER`: SMTP account user.
- `WORDPRESS_SMTP_PASSWORD`: SMTP account password.

##### PHP configuration

- `PHP_ENABLE_OPCACHE`: Enable OPcache for PHP scripts. Default: **yes**
- `PHP_EXPOSE_PHP`: Enables HTTP header with PHP version. No default.
- `PHP_MAX_EXECUTION_TIME`: Maximum execution time for PHP scripts. No default.
- `PHP_MAX_INPUT_TIME`: Maximum input time for PHP scripts. No default.
- `PHP_MAX_INPUT_VARS`: Maximum amount of input variables for PHP scripts. No default.
- `PHP_MEMORY_LIMIT`: Memory limit for PHP scripts. Default: **256M**
- `PHP_POST_MAX_SIZE`: Maximum size for PHP POST requests. No default.
- `PHP_UPLOAD_MAX_FILESIZE`: Maximum file size for PHP uploads. No default.

#### Examples

##### SMTP configuration using a Gmail account

This would be an example of SMTP configuration using a Gmail account:

- Modify the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-wordpress/blob/master/docker-compose.yml) file present in this repository:

    ```yaml
      wordpress:
        ...
        environment:
          - WORDPRESS_DATABASE_USER=bn_wordpress
          - WORDPRESS_DATABASE_NAME=bitnami_wordpress
          - ALLOW_EMPTY_PASSWORD=yes
          - WORDPRESS_SMTP_HOST=smtp.gmail.com
          - WORDPRESS_SMTP_PORT=587
          - WORDPRESS_SMTP_USER=your_email@gmail.com
          - WORDPRESS_SMTP_PASSWORD=your_password
      ...
    ```

- For manual execution:

    ```console
    $ docker run -d --name wordpress -p 80:8080 -p 443:8443 \
      --env WORDPRESS_DATABASE_USER=bn_wordpress \
      --env WORDPRESS_DATABASE_NAME=bitnami_wordpress \
      --env WORDPRESS_SMTP_HOST=smtp.gmail.com \
      --env WORDPRESS_SMTP_PORT=587 \
      --env WORDPRESS_SMTP_USER=your_email@gmail.com \
      --env WORDPRESS_SMTP_PASSWORD=your_password \
      --network wordpress-tier \
      --volume /path/to/wordpress-persistence:/bitnami \
      bitnami/wordpress-nginx:latest
    ```

##### Connect WordPress container to an existing database

The Bitnami WordPress container supports connecting the WordPress application to an external database. This would be an example of using an external database for WordPress.

- Modify the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-wordpress/blob/master/docker-compose.yml) file present in this repository:

    ```diff
       wordpress:
         ...
         environment:
    -      - WORDPRESS_DATABASE_HOST=mariadb
    +      - WORDPRESS_DATABASE_HOST=mariadb_host
           - WORDPRESS_DATABASE_PORT_NUMBER=3306
           - WORDPRESS_DATABASE_NAME=wordpress_db
           - WORDPRESS_DATABASE_USER=wordpress_user
    -      - ALLOW_EMPTY_PASSWORD=yes
    +      - WORDPRESS_DATABASE_PASSWORD=wordpress_password
         ...
    ```

- For manual execution:

    ```console
    $ docker run -d --name wordpress\
      -p 8080:8080 -p 8443:8443 \
      --network wordpress-network \
      --env WORDPRESS_DATABASE_HOST=mariadb_host \
      --env WORDPRESS_DATABASE_PORT_NUMBER=3306 \
      --env WORDPRESS_DATABASE_NAME=wordpress_db \
      --env WORDPRESS_DATABASE_USER=wordpress_user \
      --env WORDPRESS_DATABASE_PASSWORD=wordpress_password \
      --volume wordpress_data:/bitnami/wordpress \
      bitnami/wordpress-nginx:latest
    ```

In case the database already contains data from a previous WordPress installation, you need to set the variable `WORDPRESS_SKIP_BOOTSTRAP` to `yes`. Otherwise, the container would execute the installation wizard and could modify the existing data in the database. Note that, when setting `WORDPRESS_SKIP_BOOTSTRAP` to `yes`, values for environment variables such as `WORDPRESS_USERNAME`, `WORDPRESS_PASSWORD` or `WORDPRESS_EMAIL` will be ignored. Make sure that, in this imported database, the table prefix matches the one set in `WORDPRESS_TABLE_PREFIX`.

## WP-CLI tool

The Bitnami WordPress container includes the command line interface **wp-cli** that can help you to manage and interact with your WP sites. To run this tool, please note you need use the proper system user, **daemon**.

This would be an example of using **wp-cli** to display the help menu:

* Using `docker-compose` command:

```console
$ docker-compose exec wordpress wp help
```

* Using `docker` command:

```console
$ docker exec wordpress wp help
```

Find more information about parameters available in the tool in the [official documentation](https://make.wordpress.org/cli/handbook/config/).

## Logging

The Bitnami WordPress Docker image sends the container logs to `stdout`. To view the logs:

```console
$ docker logs wordpress
```

Or using Docker Compose:

```console
$ docker-compose logs wordpress
```

You can configure the containers [logging driver](https://docs.docker.com/engine/admin/logging/overview/) using the `--log-driver` option if you wish to consume the container logs differently. In the default configuration docker uses the `json-file` driver.

## Maintenance

### Backing up your container

To backup your data, configuration and logs, follow these simple steps:

#### Step 1: Stop the currently running container

```console
$ docker stop wordpress
```

Or using Docker Compose:

```console
$ docker-compose stop wordpress
```

#### Step 2: Run the backup command

We need to mount two volumes in a container we will use to create the backup: a directory on your host to store the backup in, and the volumes from the container we just stopped so we can access the data.

```console
$ docker run --rm -v /path/to/wordpress-backups:/backups --volumes-from wordpress busybox \
  cp -a /bitnami/wordpress /backups/latest
```

### Restoring a backup

Restoring a backup is as simple as mounting the backup as volumes in the containers.

For the MariaDB database container:

```diff
 $ docker run -d --name mariadb \
   ...
-  --volume /path/to/mariadb-persistence:/bitnami/mariadb \
+  --volume /path/to/mariadb-backups/latest:/bitnami/mariadb \
   bitnami/mariadb:latest
```

For the WordPress container:

```diff
 $ docker run -d --name wordpress \
   ...
-  --volume /path/to/wordpress-persistence:/bitnami/wordpress \
+  --volume /path/to/wordpress-backups/latest:/bitnami/wordpress \
   bitnami/wordpress:latest
```

### Upgrade this image

Bitnami provides up-to-date versions of MariaDB and WordPress, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the WordPress container. For the MariaDB upgrade see https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

The `bitnami/wordpress-nginx:latest` tag always points to the most recent release. To get the most recent release you can simple repull the `latest` tag from the Docker Hub with `docker pull bitnami/wordpress-nginx:latest`. However it is recommended to use [tagged versions](https://hub.docker.com/r/bitnami/wordpress-nginx/tags/).

#### Step 1: Get the updated image

```console
$ docker pull bitnami/wordpress-nginx:latest
```

#### Step 2: Stop the running container

Stop the currently running container using the command

```console
$ docker-compose stop wordpress
```

#### Step 3: Take a snapshot of the application state

Follow the steps in [Backing up your container](#backing-up-your-container) to take a snapshot of the current application state.

#### Step 4: Remove the currently running container

Remove the currently running container by executing the following command:

```console
docker-compose rm -v wordpress
```

#### Step 5: Run the new image

Update the image tag in `docker-compose.yml` and re-create your container with the new image:

```console
$ docker-compose up -d
```

## Customize this image

The Bitnami WordPress with NGINX Docker image is designed to be extended so it can be used as the base image for your custom web applications.

### Extend this image

Before extending this image, please note there are certain configuration settings you can modify using the original image:

- Settings that can be adapted using environment variables. For instance, you can change the ports used by NGINX for HTTP and HTTPS, by setting the environment variables `NGINX_HTTP_PORT_NUMBER` and `NGINX_HTTPS_PORT_NUMBER` respectively.
- [Adding custom server blocks](https://github.com/bitnami/bitnami-docker-nginx#adding-custom-server-blocks).
- [Replacing the 'nginx.conf' file](https://github.com/bitnami/bitnami-docker-nginx#full-configuration).
- [Using custom SSL certificates](https://github.com/bitnami/bitnami-docker-nginx#using-custom-ssl-certificates).

If your desired customizations cannot be covered using the methods mentioned above, extend the image. To do so, create your own image using a Dockerfile with the format below:

```Dockerfile
FROM bitnami/wordpress-nginx
## Put your customizations below
...
```

Here is an example of extending the image with the following modifications:

- Install the `vim` editor
- Modify the NGINX configuration file
- Modify the ports used by NGINX

```Dockerfile
FROM bitnami/wordpress-nginx
LABEL maintainer "Bitnami <containers@bitnami.com>"

## Change user to perform privileged actions
USER 0
## Install 'vim'
RUN install_packages vim
## Revert to the original non-root user
USER 1001

## Modify 'worker_connections' on NGINX config file to '512'
RUN sed -i -r "s#(\s+worker_connections\s+)[0-9]+;#\1512;#" /opt/bitnami/nginx/conf/nginx.conf

## Modify the ports used by NGINX by default
# It is also possible to change these environment variables at runtime
ENV NGINX_HTTP_PORT_NUMBER=8181
ENV NGINX_HTTPS_PORT_NUMBER=8143
EXPOSE 8181 8143
```

Based on the extended image, you can update the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-wordpress-nginx/blob/master/docker-compose.yml) file present in this repository to add other features:

```diff
   wordpress:
-    image: bitnami/wordpress-nginx:latest
+    build: .
     ports:
-      - '80:8080'
-      - '443:8443'
+      - '80:8181'
+      - '443:8143'
     environment:
+      - PHP_MEMORY_LIMIT=512m
     ...
```

## Notable Changes

### 5.7.1-debian-10-r22

- The size of the container image has been decreased.
- The configuration logic is now based on Bash scripts in the *rootfs/* folder.
- Multisite support was added via `WORDPRESS_ENABLE_MULTISITE` and related environment variables.
- Plugins can be installed and activated on the first deployment via `WORDPRESS_PLUGINS`.
- Added support for limiting auto-updates to WordPress core via `WORDPRESS_AUTO_UPDATE_LEVEL`. In addition, auto-updates have been disabled by default. To update WordPress core, we recommend to swap the container image version for your deployment instead of using the built-in update functionality.
- This image now supports connecting to MySQL and MariaDB databases securely via SSL.

### 5.3.2-debian-10-r30

- The WordPress with NGINX container has been migrated to a "non-root" user approach. Previously the container ran as the `root` user and the NGINX daemon was started as the `daemon` user. From now on, both the container and the NGINX daemon run as user `1001`. You can revert this behavior by changing `USER 1001` to `USER root` in the Dockerfile.
- Consequences:
  - The HTTP/HTTPS ports exposed by the container are now `8080/8443` instead of `80/443`.
  - Backwards compatibility is not guaranteed when data is persisted using docker or docker-compose. We highly recommend migrating the WP site by exporting its content, and importing it on a new WordPress container. In the links below you'll find some alternatives:
    - [Migrate WordPress using All-in-One WP Migration plugin](https://docs.bitnami.com/general/how-to/migrate-wordpress/)
    - [Migrate WordPress using VaultPress](https://vaultpress.com/)
  - No writing permissions will be granted on `wp-config.php` by default.

### 5.2.1-debian-9-r8 and 5.2.1-ol-7-r8

- This image has been adapted so it's easier to customize. See the [Customize this image](#customize-this-image) section for more information.
- The PHP configuration volume (`/bitnami/php`) has been deprecated, and support for this feature will be dropped in the near future. Until then, the container will enable the PHP configuration from that volume if it exists. By default, and if the configuration volume does not exist, the configuration files will be regenerated each time the container is created. Users wanting to apply custom PHP configuration files are advised to mount a volume for the configuration at `/opt/bitnami/php/conf`, or mount specific configuration files individually.
- Enabling custom NGINX certificates by placing them at `/opt/bitnami/nginx/certs` has been deprecated, and support for this functionality will be dropped in the near future. Users wanting to enable custom certificates are advised to mount their certificate files on top of the preconfigured ones at `/certs`.

## Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-wordpress-nginx/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-wordpress-nginx/pulls) with your contribution.

## Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-wordpress-nginx/issues/new). Be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

### Community supported solution

Please, note this asset is a community-supported solution. This means that the Bitnami team is not actively working on new features/improvements nor providing support through GitHub Issues. Any new issue will stay open for 20 days to allow the community to contribute, after 15 days without activity the issue will be marked as stale being closed after 5 days.

The Bitnami team will review any PR that is created, feel free to create a PR if you find any issue or want to implement a new feature.

New versions and releases cadence are not going to be affected. Once a new version is released in the upstream project, the Bitnami container image will be updated to use the latest version, supporting the different branches supported by the upstream project as usual.

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
