## Prerequisites.

First, install Docker Community Edition. See instructions for [MacOS](https://docs.docker.com/docker-for-mac/install/), [Windows](https://docs.docker.com/docker-for-windows/install/), or your preferred flavor of Linux ([CentOS, RHEL, Amazon Linux](https://docs.docker.com/install/linux/docker-ce/centos/), [Debian](https://docs.docker.com/install/linux/docker-ce/debian/), [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/), [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)). After that, install [Docker Compose](https://docs.docker.com/compose/install/).

## Set Up

### Extract the Alpha

Extract the contents of the Linux alpha build (not Windows or Mac!) to your `/app` directory of this repo. If your extraction utility placed the contents into a sub-folder, make sure the contents (dist, lib, locales, node_modules, etc.) are directly inside the `/app` directory.

### Update the Docker Compose (optional).

For maximum performance, this setup assumes that the only directory you want to make updates to and sync with the Foundry node container are the `/public/worlds` directory contents. However, you can sync other directories (such as `/public/systems`), if you choose. In order to do that, add the additional directories to `/public` (not `/app/public`), and add additional volume to the `node` service in the `docker-compose.yml` file. For instance:

```yaml
    volumes:
      - ./public/worlds:/app/public/worlds
      - ./public/systems:/app/public/systems
```

Proceed with caution if syncing systems in this way; if the alpha has additional system updates, you'll need to make sure they're copied over to your `/public` directory.

### Run docker-compose.yml

To start the containers, you'll need to run the following command from this repo's main directory:

```
docker-compose up -d
```

The `-d` option is for detached mode, but you can also leave the option off if you want to see log files. If you start in detached mode, you can stop the containers with `docker-compose down`.

## Updating Versions

Whenever a new version of Foundry is released, extract its contents to the `/app` directory just like before. Because the app directory is baked into the node container that we're using for Foundry, you'll want to make sure to rebuild the node container. Run the following the first time after starting the containers following an update:

```
docker-compose up -d --build
```

It's also probably a good idea to update the tag number for the node service's image in the `docker-compose.yml`. For example, this repo is currently assuming you're using alpha version 0.0.3:

```yaml
version: '3'

services:
  # web server
  node:
    build:
      context: ./
      dockerfile: Dockerfile-foundrynode
    # Update the version number below, as needed.
    image: foundrynode:0.0.3
    container_name: foundry_node
    ports:
      - "30000:30000"
    volumes:
      - ./public/worlds:/app/public/worlds
  filebrowser:
    image: filebrowser/filebrowser
    container_name: foundry_filebrowser
    ports:
      - "8080:80"
    volumes:
      - ./public:/srv
      - ./config/filebrowser/config.json:/config/json
      - ./data/database.db:/database.db
```

## Access Foundry in the browser!

To pull up Foundry, go to `http://localhost:30000`, or `http://yourdomain.com:30000` if you're using a server with a real domain.

To pull up the basic File Browser service, go to `http://localhost:8080`, or `http://yourdomain.com:8080` if you're using a server with a real domain. The file browser is pretty bare bones, but it covers simple operations and does allow you to preview images and audio files. Files will be uploaded directly into your worlds directory, so come up with whatever structure works best for you! I personally use a `custom_assets` directory inside my specific world's directory, such as `/worlds/WORLDNAME/custom_assets`, and then that's further divided into maps, tokens, and music.