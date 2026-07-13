# trial-wordpress

Trial or micro-service unit of WordPress.

## Usage

### Start using docker compose

Manage service stack with [Docker Compose](https://docs.docker.com/compose/).

```bash
# Initiate .env file
cp .env.example .env
# Start services
docker compose up -d
```

Update existing composed containers:

```bash
docker compose pull && \
docker compose down && \
docker compose up -d
```

### Start services individually

```bash
docker run --network=trial-mysql_backend -p 2000:80 --restart=always -d wordpress
```
