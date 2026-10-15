# hosted-postal-server

> **Status: WIP.** Stack runs locally, SMTP AUTH verified. Still pending: verified sending domain, end-to-end delivery test, TLS cert, production DNS.

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

Web UI: `http://localhost:5001` (5000 is macOS AirPlay). SMTP: `localhost:25` or `localhost:587`. DB inspect: `localhost:3307` (host 3306 is your machine mysqld).

## Config (MariaDB only — Postal has no Postgres support)

| Var | Default | Purpose |
|-----|---------|---------|
| `DATABASE_HOST` | `mariadb` | `host.docker.internal` for machine MariaDB |
| `DATABASE_PORT` | `3306` | |
| `DATABASE_NAME` | `postal` | main DB + message DB prefix |
| `DATABASE_USERNAME` | `root` | must exist on the DB host |
| `DATABASE_PASSWORD` | `postal` | Must match `postal.yml` |
| `POSTAL_VERSION` | `3.3.7` | Pin exact version; bump + `postal upgrade` to update |

`./config:/config` must contain `postal.yml` + `signing.key`.

## Send mail from your app (SMTP)

1. Login to the Web UI → create organization → create mail server.
2. Add your sending domain, publish its SPF/DKIM/return-path DNS records.
3. Mail server → Credentials → add SMTP credential → copy key + secret.
4. Put these in your app's `.env`:

```bash
SMTP_HOST=localhost          # your postal domain in production
SMTP_PORT=587                # 25 also works; both hit the same server locally
SMTP_USER=<smtp credential key>
SMTP_PASSWORD=<smtp credential secret>
SMTP_FROM=hello@example.com  # must be on a verified sending domain
SMTP_TLS=false               # local has no TLS; true only with a cert in postal.yml
```

## Useful

```bash
docker compose ps
docker compose logs -f web smtp worker
docker compose run --rm runner postal upgrade  # after bumping POSTAL_VERSION
```

Production: add Caddy/HTTPS, real DNS (SPF/DKIM/return-path), backups of `mariadb-data` + `config/`.
