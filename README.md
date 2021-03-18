# Media Server Compose Notes

## Build/Refresh

```bash
docker-compose pull
docker-compose up --detach
docker image prune --force
```

## Setup

Need a user and group named `docker`, and to update the .env with the PUID and PGID.
Need a config and storage folder structure.

## Maintain

Make a cron job that executes the build script.