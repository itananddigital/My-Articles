
We will setup the following:

- Install docker and docker compose v2 on linux server.
- Install traefik service for internal load balancer and letsencrypt.
- Install MariaDB with containers.
- Persistent Volume for site data

Explanation:

Single instance of **Traefik** will be installed and act as internal loadbalancer for multiple benches and sites hosted on the server. It can also load balance other applications along with frappe benches, e.g. wordpress, metabase, etc. We only expose the ports `80` and `443` once with this instance of traefik. Traefik will also take care of letsencrypt automation for all sites installed on the server. _Why choose Traefik over Nginx Proxy Manager?_ Traefik doesn't need additional DB service and can store certificates in a json file in a volume.

Single instance of **MariaDB** will be installed and act as database service for all the benches/projects installed on the server.

Each instance of ERPNext project (bench) will have its own redis, socketio, gunicorn, nginx, workers and scheduler. It will connect to internal MariaDB by connecting to MariaDB network. It will expose sites to public through Traefik by connecting to Traefik network.

### Install Docker

Easiest way to install docker is to use the [convenience script](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script).

```shell
curl -fsSL https://get.docker.com | bash
```

Note: The documentation assumes Ubuntu LTS server is used. Use any distribution as long as the docker convenience script works. If the convenience script doesn't work, you'll need to install docker manually.

### Install Compose V2

Refer [original documentation](https://docs.docker.com/compose/cli-command/#install-on-linux) for updated version.

```shell
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```

### Prepare

Clone `frappe_docker` repo for the needed YAMLs and change the current working director of you shell to the cloned repo.

```shell
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

Create configuration and resources directory

```shell
mkdir ~/gitops
```

The `~/gitops` directory will store all the resources that we use for setup. We will also keep the environment files in this directory as there will be multiple projects with different environment variables. You can create a private repo for this directory and track the changes there.

### Install Traefik

Basic Traefik setup using docker compose.

Create a file called `traefik.env` in `~/gitops`

```shell
echo 'TRAEFIK_DOMAIN=traefik.example.com' > ~/gitops/traefik.env
echo 'EMAIL=admin@example.com' >> ~/gitops/traefik.env
echo 'HASHED_PASSWORD='$(openssl passwd -apr1 YOUR_PASSWORD | sed 's/\$/\\\$/g') >> ~/gitops/traefik.env
```

Deploy the traefik container with letsencrypt SSL

```shell
docker compose --project-name traefik \
  --env-file ~/gitops/traefik.env \
  -f overrides/compose.traefik.yaml \
  -f overrides/compose.traefik-ssl.yaml up -d
```

This will make the traefik dashboard available on `traefik.example.com` and all certificates will reside in `/data/traefik/certificates` on host filesystem.

For LAN setup deploy the traefik container without overriding `overrides/compose.traefik-ssl.yaml`.

### Install MariaDB

Basic MariaDB setup using docker compose.

Create a file called `mariadb.env` in `~/gitops`

```shell
echo "DB_PASSWORD=YOUR_PASSWORD" > ~/gitops/mariadb.env
```

```shell
docker compose --project-name mariadb --env-file ~/gitops/mariadb.env -f overrides/compose.mariadb-shared.yaml up -d
```

### Create a Persistent Volume

Create a persistent volume to store your site data

```shell
$ sudo mkdir -p /mnt/data/sites
```
Set the required permissions:

```shell
$ sudo chmod 775 -R /mnt/data
$ sudo chown -R $USER:docker /mnt/data
```
Now, create the Docker volume for your sites:

```shell
$ docker volume create --driver local \
--opt type=none \
--opt device=/mnt/data/sites \
--opt o=bind sites
```
