# Scylla Docker Deployment

A Docker-based deployment for the Scylla backend and frontend applications.

## Quickstart

Follow these steps to set up and run Scylla using Docker:

### 1. Clone Scylla repos

Create a folder for Scylla-related repos:

```bash
mkdir scylla && cd scylla
```

Clone Scylla repos:

```bash
git clone https://github.com/toadmaninteractive/scylla-backend
git clone https://github.com/toadmaninteractive/scylla-frontend
git clone https://github.com/toadmaninteractive/scylla-docker
```

### 2. Build Scylla backend image

Open `scylla-backend` dir and run:

```bash
docker build -t scylla-backend:latest .
```

### 3. Build Scylla frontend image

Open `scylla-frontend` dir and run:

```bash
docker build -t scylla-frontend:latest --build-arg BUILD_ENV=production .
```

### 4. Create dedicated Docker network

```bash
docker network create scylla
```

### 5. Configure Scylla

Open `scylla-docker` dir and create configuration files from examples:

```bash
cp .env.example .env
cp config.yaml.example config.yaml
```

Edit the config.yaml file:
- Fill in your LDAP settings (LDAP server address and bind user CN / password)

### 6. Prepare database

Run this command from the `scylla-docker` directory to initialize the database (once):

```bash
docker compose run \
    --name scylla-migrator \
    --user root \
    --no-deps \
    --rm \
    --entrypoint "bash -c 'cd /migrations && ./graphile-migrate run --rootDatabase migrations/00-init.sql && ./graphile-migrate reset --erase'" \
    scylla-backend
```

Run this command from the `scylla-docker` directory to migrate the database (incremental):

```bash
docker compose run \
    --name scylla-migrator \
    --user root \
    --no-deps \
    --rm \
    --entrypoint "bash -c 'cd /migrations && ./graphile-migrate migrate'" \
    scylla-backend
```

### 7. Run Docker Compose

```bash
docker compose up -d
```

To stop Scylla and related services (except Traefik) run this command from the `scylla-docker` directory:

```bash
docker compose down
```

## Accessing Scylla in a browser

After completing the Quickstart guide, Scylla will be running in Docker containers.
To access the web interface through a browser, you'll need to set up a reverse proxy.

We provide two recommended approaches:

### Option 1: Local Proxy Setup
Configure a local proxy server such as **nginx** to route traffic to the Scylla containers.

### Option 2: Traefik Proxy (Recommended for Production)
We provide a pre-configured [Traefik](https://github.com/traefik/traefik) setup for production deployments.
Traefik automatically handles SSL termination, load balancing, and routing.

#### Prerequisites for Traefik
- **Registered domain name** pointing to your server
- **SSL certificates** (certificate chain and private key)

#### Traefik Configuration Steps

1. **Prepare SSL certificates**
- Save your certificate chain as `cert_key_chain.crt`
- Save your private key as `private.key`
- Place both files in the traefik/certs directory

2. **Configure your domain**
- Edit `traefik/conf/dynamic/scylla.yml`
- Replace `scylla.yourcompany.com` with your actual domain name

3. **Start Traefik**

Run this command from the `docker/traefik` directory:

```bash
docker compose up -d
```

4. **Stop Traefik**

Run this command from the `docker/traefik` directory:

```bash
docker compose down
```

#### Verification
After starting Traefik, access Scylla at https://your-domain.com (replace with your actual domain).
The proxy automatically handles SSL encryption and redirects HTTP traffic to HTTPS.

#### Additional notes
- Ensure your firewall allows traffic on ports 80 (HTTP) and 443 (HTTPS)
- DNS records should point to your server's public IP address
- Certificate files must be in PEM format
- Traefik automatically monitors container changes and updates routing accordingly
