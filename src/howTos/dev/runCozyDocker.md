---
title: Run your application inside a Cozy in Docker
summary: Develop using a local development Cozy
---

Make sure your application is built into `~/cozy/myapp` (it should have an `index.html` and a `manifest.webapp` files), otherwise it will not work. As an example, for the [Drive application](https://github.com/cozy/cozy-drive/), it should be `~/cozy/drive/build`.

Also make sure you have pulled the last version of the Docker image `cozy/cozy-app-dev`:

```sh
docker pull cozy/cozy-app-dev
```

!!! warning ""
    On GNU/Linux, according [to the documentation](https://docs.docker.com/engine/installation/linux/linux-postinstall/): « The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user root and other users can only access it using sudo. If you don’t want to use sudo when you use the docker command, create a Unix group called docker and add users to it. Be warned that the docker group grants privileges equivalent to the root user. You should have a look at [Docker’s documentation on security](https://docs.docker.com/engine/security/security/).

## Ephemeral instance

To run a ephemeral instance, on the `~/cozy/myapp` directory, you have to mount the folder inside the server to `/data/cozy-app`. This is what make the application available on the server:

```sh
$ docker run --rm -it \
    -p 8080:8080 \
    -p 8025:8025 \
    --name cozydev \
    -v "~/cozy/myapp":/data/cozy-app \
    cozy/cozy-app-dev
```

!!! Note "Commands explaination"
    - `--rm` will delete the server when you stop it. This prevent Docker from keeping a lot of unused stopped images
    - `-it` allow to attach an interactive terminal, so you’ll be able to use the command line inside the server
    - `-p 8080:8080`: the server listens on port 8080 on the virtual machine. We forward this port to the same port on your local machine. To use another local port, for example 9090, use `-p 9090:8080`
    - `-p 8025:8025` : Cozy requires a mail server. In the development image, we don’t use a real email server, but a software that can display the sent messages. Just point your browser to `http://cozy.tools:8025/` to display the messages sent by the server
    - `--name cozydev` name the running virtual machine `cozydev`, so you can easily refer to it from other Docker commands. For example, if you want to connect to a shell inside the server, you can use `docker exec cozydev -it /bin/bash`
    - `-v $(pwd):/data/cozy-app` this mount the current folder, where your application leaves, inside the server. This is what make the application available on the server

## With data persistence

If you want to persist data, you have to mount two folders from the virtual server to local folders: `/usr/local/couchdb/data` (database) and `/data/cozy-storage` (the virtual filesystem). This can be achieved by adding to the two command lines with `-v` which will store the server’s data into `~/cozy/data/`.

```sh
$ docker run --rm -it \
    -p 8080:8080 \
    -p 8025:8025 \
    --name cozydev \
    -v "~/cozy/myapp":/data/cozy-app \
    -v "~/cozy/data/db":/usr/local/couchdb/data \
    -v "~/cozy/data/storage":/data/cozy-storage \
    cozy/cozy-app-dev
```

## Run with a custom stack config file

The `cozy-stack` config file can be useful to change the log level for example. You have to load it inside `/etc/cozy/cozy.yaml` like this:

```sh
$ docker run --rm -it \
    -p 8080:8080 \
    -p 8025:8025 \
    --name cozydev \
    -v "~/cozy/myapp":/data/cozy-app \
    -v "~/cozy/cozy.yaml":/etc/cozy/cozy.yaml \
    cozy/cozy-app-dev
```

You can learn more about the `cozy-stack` config file in [its dedicated documentation](/cozy-stack/config/)

## Run two applications watched locally

You can install more than one application into the development server, for example to test communication between applications. In order to achieve this, you have to mount the folder where your application leaves into subfolders of `/data/cozy-apps` like this:

```sh
$ docker run --rm -it \
    -p 8080:8080 \
    -p 8025:8025 \
    --name cozydev \
    -v "~/cozy/appone":/data/cozy-app/appone \
    -v "~/cozy/apptwo":/data/cozy-app/apptwo \
    cozy/cozy-app-dev
```

You’ll access the applications by connecting to `http://appone.cozy.tools:8080/` and `http://apptwo.cozy.tools:8080`.

## Mailhog to catch e-mails and CouchDB access

A [MailHog](https://github.com/mailhog/MailHog) is running inside docker to
catch emails. You can view the emails sent by the stack in a web interface on
http://cozy.tools:8025/

You can also expose the couchdb port (listening in the container on 5984) in
order to access its admin page. For instance add `-p 5984:5984` to access to the
admin interface on `http://cozy.tools:5984/_utils`.
