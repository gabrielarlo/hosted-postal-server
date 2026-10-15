# hosted-postal-server

Self-hosted [Postal](https://docs.postalserver.io/) v3 (open-source mail delivery) via Docker Compose: `web` + `smtp` + `worker` + MariaDB.

## Prereqs

Docker + Compose plugin, DNS + port 25 open, 4GB RAM / 2 CPU min.

## Quickstart

```bash
mkdir -p config && openssl genrsa -out config/signing.key 2048
# create config/postal.yml: set web_hostname, smtp_hostname, DB password, DB host: mariadb
docker compose up -d mariadb
docker compose run --rm runner postal initialize
docker compose up -d
docker compose run --rm runner postal make-user
```

Web UI: `http://localhost:5000`. SMTP: `localhost:25`.

## Config

| Var | Default | Purpose |
|-----|---------|---------|
| `MARIADB_ROOT_PASSWORD` | `postal` | Must match `postal.yml` |
| `POSTAL_VERSION` | `3` | Pin e.g. `3.3.4` for prod |

`./config:/config` must contain `postal.yml` + `signing.key`.

## Useful

```bash
docker compose ps
docker compose logs -f web smtp worker
docker compose run --rm runner postal upgrade  # after bumping POSTAL_VERSION
```

Production: add Caddy/HTTPS, real DNS (SPF/DKIM/return-path), backups of `mariadb-data` + `config/`.
