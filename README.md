# infra-blocks-wordpress

[![Blocks](https://img.shields.io/badge/infra--blocks-index-555?style=flat-square)](https://github.com/kamaslau/infra-blocks)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
![Repository size](https://img.shields.io/github/repo-size/kamaslau/infra-blocks-wordpress?color=56BEB8)

A unit, not a demo: one service, one port, one documented credential, disposable with a single command.

Typical use: a client wants to edit their own pages without opening a ticket, and the site has to live on a host you already administer.

[WordPress](https://wordpress.org/documentation/) on MySQL 8.4. For the database on its own, see [infra-blocks-mysql](https://github.com/kamaslau/infra-blocks-mysql).

## Service URL

- WordPress `http://localhost:80`

Ports can be changed in `.env`; see Usage.

## Default user

There is none until you finish the installer. The first visit asks for a site title, an admin account and a password, and writes all three into the database — nothing in this repository predates that.

| Setting | Default |
| ------- | ------- |
| Database | `wordpress` |
| Database user | `wordpress` |
| Database password | `wordpress` |
| Root password | `rootpassword` |

Defaults are for local use only. The installer asks for the database host as `db`, which is the service name in `compose.yml`.

## Usage

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

The two containers have to share a network, and the database has to answer before WordPress starts.

```bash
docker network create trial-wordpress_backend

docker run -d --restart unless-stopped --name wordpress-db \
  --network trial-wordpress_backend \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wordpress \
  -e MYSQL_PASSWORD=wordpress \
  mysql:8.4

docker run -d --restart unless-stopped --name wordpress \
  --network trial-wordpress_backend \
  -p 80:80 \
  -e WORDPRESS_DB_HOST=wordpress-db \
  -e WORDPRESS_DB_USER=wordpress \
  -e WORDPRESS_DB_PASSWORD=wordpress \
  -e WORDPRESS_DB_NAME=wordpress \
  wordpress:7.0-php8.5
```

## Verify

```bash
curl -sI http://localhost:80 | head -1
```

`HTTP/1.1 302 Found` is the answer you want: WordPress is running and pointing the browser at the installer. A `500` instead means the container is up and the database connection is not — `docker compose logs wordpress` says `Error establishing a database connection` when that is the cause.

After the installer, `curl -s http://localhost:80 | grep -o '<title>[^<]*'` returns the site title you typed, which proves the database is being read as well.

## Why this shape

The database is a separate service, not a second database inside the WordPress container, because that is the shape a handover takes: `docker compose down` keeps the posts, `down -v` removes both volumes and the install with them. Two volumes rather than one, so a database dump can be taken without the uploads directory.

`depends_on: condition: service_healthy` with a `mysqladmin ping` check is there because MySQL accepts a socket seconds before it accepts queries, and WordPress does not retry the first connection — it prints the connection error on the install page and stays there.

The database container is named `wordpress-db` rather than `mysql`, so this unit and [infra-blocks-mysql](https://github.com/kamaslau/infra-blocks-mysql) can run on the same machine without one of them refusing to start on a name collision.

The image tag is pinned (`7.0-php8.5`) instead of `latest`, because a WordPress minor upgrade can rewrite the schema without asking, and `docker compose pull` would then be the command that broke the site. `latest` belongs on a site nobody has to hand over.

## Notes for deployments in China

Both images pull from Docker Hub and need a registry mirror on most mainland hosts. WordPress also calls `api.wordpress.org` for core and plugin updates; on a host without a route to it, the dashboard shows update failures that are network problems, not permissions.

Uploads live in the `wordpress_data` volume, so moving the site is `docker compose down` followed by a volume copy, not a file-by-file `scp`.

---

Part of [infrastructure blocks](https://github.com/kamaslau/infra-blocks) · ← back to index
