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

Get the PUID and PGID with `id docker` and then update `.env` variables.

### Container Settings

#### CalibreWeb

You need to create a fresh calibre DB. The CalibreWeb container GUI can't make a "library", but it has the tools to do it via the CLI.

1. Initialize the DB:
   ```bash
   docker-compose exec calibre-web calibredb restore_database --really-do-it --with-library /books
   ```

Make sure that permissions are 755 on host machien for all storage folders.

#### Jdownloader

To reset the login credentials, I didn't want to stick them into the compose yaml so here's how you update it from the cli:

```bash
docker-compose exec jdownloader configure email@email.com password
docker-compose restart jdownloader
```

## Maintain

Make a cron job that executes the build script.cd /
