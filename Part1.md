Step 1: Access Your Server and Update
Before we begin, log in to your Ubuntu 22.04 server and ensure your system is up-to-date by running the following commands:


COPY
$ su - your-username
$ sudo apt update
Step 2: Install Docker
Docker allows you to run containers easily. Install Docker and its prerequisites with the following commands:


COPY
$ sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
$ sudo apt install docker-ce
After installing Docker, add your system user to the Docker group to enable Docker commands without using sudo:


COPY
$ sudo usermod -aG docker $USER
$ newgrp docker
Start Docker and set it to launch at boot:


COPY
$ sudo systemctl start docker && sudo systemctl enable docker
Step 3: Installation and Setup
Clone the Repository
To get started with your containerized applications, clone the Frappe Docker repository and navigate to the directory:


COPY
$ git clone https://github.com/frappe/frappe_docker
$ cd frappe_docker
Create a Directory for Resources and Configs
Now, create a directory to store your resources and configurations:


COPY
$ mkdir ~/gitops
Ensure that OpenSSL is installed on your server:


COPY
$ sudo apt install openssl
Install Traefik Ingress
To set up Traefik, you'll need to export some environment variables for your domain and Letsencrypt notification email. Replace your-domain.com and your-email@example.com with your domain and email:


COPY
echo 'TRAEFIK_DOMAIN=your-domain.com' > ~/gitops/traefik.env
echo 'EMAIL=your-email@example.com' >> ~/gitops/traefik.env
echo 'HASHED_PASSWORD='$(openssl passwd -apr1 YourPassword | sed 's/\$/\\\$/g') >> ~/gitops/traefik.env
Once done, you can run a Traefik container using the environment file you created:


COPY
$ docker compose --project-name traefik \
  --env-file ~/gitops/traefik.env \
  -f overrides/compose.traefik.yaml up -d
If you have an FQDN and want to run the container with SSL automation (exposing port 443), use the following command:


COPY
$ docker compose --project-name traefik \
  --env-file ~/gitops/traefik.env \
  -f overrides/compose.traefik.yaml \
  -f overrides/compose.traefik-ssl.yaml up -d
Step 4: Install MariaDB Container and Create Volumes
To set up MariaDB and create volumes, follow these steps:

Create an Environment File
Create an environment file for MariaDB in the ~/gitops directory. Replace YourPassword with your desired password:


COPY
$ echo "DB_PASSWORD=YourPassword" > ~/gitops/mariadb.env
Start the MariaDB Container
Start the MariaDB container using Docker Compose:


COPY
$ docker compose --project-name mariadb --env-file ~/gitops/mariadb.env -f overrides/compose.mariadb-shared.yaml up -d
Create a Persistent Volume
Create a persistent volume to store your site data:


COPY
$ sudo mkdir -p /mnt/data/sites
Set the required permissions:


COPY
$ sudo chmod 775 -R /mnt/data
$ sudo chown -R $USER:docker /mnt/data
Now, create the Docker volume for your sites:


COPY
$ docker volume create --driver local \
--opt type=none \
--opt device=/mnt/data/sites \
--opt o=bind sites
You have successfully set up Docker, Traefik, MariaDB, and a volume for your site on your Ubuntu 22.04 server. You are now ready to deploy and manage your containerized applications.