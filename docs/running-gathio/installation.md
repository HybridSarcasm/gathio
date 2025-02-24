# Installation

Gathio can be set up to run on your own server in two ways - as a self-hosted service, or via Docker.

## Self-hosting on Linux or macOS

### Prerequisites

-   [Node.js](https://nodejs.org/en/) v18 or greater
-   [MongoDB](https://www.mongodb.com/docs/manual/administration/install-on-linux/#std-label-install-mdb-community-edition-linux)

### Ubuntu

Let's suppose we're installing on a fresh Ubuntu system.

First, let's get the code:

```
$ cd /srv/
$ sudo git clone https://github.com/lowercasename/gathio/
```

We'll need to install [pnpm](https://pnpm.io/) for this:

```
$ curl -fsSL https://get.pnpm.io/install.sh | sh -
```

pnpm installation instructions for [other systems](https://pnpm.io/installation) are available.

Now, we'll install the dependencies:

```
$ cd gathio
$ pnpm install
$ pnpm build
```

Let's copy the config file in place:

```
$ cp config/config.example.toml config/config.toml
```

We can edit this file if needed, as it contains settings which will need to be adjusted to your local setup to successfully format emails.

```
$ $EDITOR config/config.toml
```

Either way, we'll need to have MongoDB running. Follow the [MongoDB Community Edition Ubuntu instructions](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu), which are probably what you want.

Next, let's create a dedicated user:

```
$ sudo adduser --home /srv/gathio --disabled-login gathio
$ sudo chown -R gathio:gathio /srv/gathio
```

Next, we'll copy the systemd service and reload systemd

```
$ sudo cp gathio.service /etc/systemd/system/
$ sudo systemctl daemon-reload
```

Finally, we can start gathio:

```
$ sudo systemctl start gathio
```

It should now be listening on port 3000:

```
$ sudo netstat -tunap | grep LISTEN
[...]
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      952/sshd
tcp6       0      0 :::3000                 :::*                    LISTEN      5655/node
[...]
```

(this doesn't mean it's only listening on IPv6, because sockets under Linux are
dual-stack by default...)

It is now available on port 3000, and we can continue by setting up a reverse
proxy, which allows us to make it available on another port, or from another
server; and to enable TLS on the connection (see for example [Linode's guide on
the subject](https://www.linode.com/docs/web-servers/nginx/use-nginx-reverse-proxy/#configure-nginx))

## Docker

The easiest way to run Gathio using Docker is by using the provided `docker-compose` configuration.

Ensure that the `node_modules` folder does not exist in the `gathio` directory before
starting up the Docker container.

Create a directory on your system where you'll keep the Gathio configuration file. Copy the example
config file into this directory:

```
mkdir ~/docker/gathio-docker
cp config/config.example.toml ~/docker/gathio-docker/config.toml
```

Under the `volumes` section of the `docker-compose.yml` configuration, adjust the
configuration volume to match the folder you created:

```dockerfile
volumes:
    - '/home/username/docker/gathio-docker:/app/config
```

Adjust any settings in the config file, especially the MongoDB URL, which should read as follows for the standard Dockerfile config, and the email service if you want
to enable it:

```
mail_service = "nodemailer"
mongodb_url = "mongodb://mongo:27017/gathio"
```

Finally, start the Docker stack:

```
docker-compose up -d --build
```

Gathio should now be running on `http://localhost:3000`, and storing data in a Docker volume.
