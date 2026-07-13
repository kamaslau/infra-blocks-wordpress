# trial-wordpress

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
![Repository size](https://img.shields.io/github/repo-size/kamaslau/trial-wordpress?color=56BEB8)

Trial or micro-service unit of WordPress.

## Service URL

- WordPress [http://localhost](http://localhost)

## Usage

### Start using docker compose

### Start with [Docker Compose](https://docs.docker.com/compose/)

```bash
# Initiate .env file
cp .env.example .env
# Start services
docker compose up -d
```

Update existing composed containers with latest images:

```bash
docker compose pull && \
docker compose down && \
docker compose up -d
```

### Start services individually

```bash
docker run --network=trial-mysql_backend -p 2000:80 --restart=always -d wordpress
```
