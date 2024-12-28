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

[docker-jdownloader](https://github.com/jaymoulin/docker-jdownloader)

To reset the login credentials, I didn't want to stick them into the compose yaml so here's how you update it from the cli:

```bash
docker-compose exec jdownloader configure email@email.com password
docker-compose restart jdownloader
```

**May need to perform the password setting while container is restarting, like do a restart, do the password, and watch logs**

```bash
docker-compose restart jdownloader
docker-compose exec jdownloader configure email@email.com password
docker-compose logs -f --tail=50 jdownloader
```

**This seems to lose all history**

Currently have a weird issue with not being able to map the configs correctly.

```yaml
volumes:
  - ${PATH_CONFIG}/jdownloader:/opt/JDownloader/cfg
  # - ${PATH_CONFIG}/jdownloader:/opt/JDownloader/app/cfg # But this breaks things
  - ${PATH_JDOWN}:/opt/JDownloader/Downloads
```

I have a typo in the config path, but when I tried to correct it to the right one in the doc I get a bunch of permission issues. All of the files seem to be getting made as `root` instead of the uid I provide. Only things with the uid:gid I give are the volumes I define. So when I set the right path to the config, it's my uid:gid and that breaks everything as it is needing to be root.

For now, I just won't preserve the configs I guess. And then just try to be careful.

#### TubeSync

I was looking for a way to back up some youtube videos for long car rides without decent signals. This tool looks like a decent stopgap until I build something more custom.

To get things to load into Plex correctly, we need a special _Media Format_:

```bash
# Original
{yyyy_mm_dd}_{source}_{title}_{key}_{format}.{ext}

# New?
{source_full} - {yyyy_mm_dd} - {title_full} [{key}] [{format}].{ext}
## Channel Name - 2021-01-31 - Some title [SoMeUnIqUeId] [720p-avc1-mp4a].mkv
```

Also, you need to install the YouTube-Agent.bundle to get metadata.

I'm not super thrilled with the encoding on these files so I want to convert over to x265. There's a way to do it with ffmpeg, so I want to test out this docker approach.

```bash
file_to_convert="input_name"
docker run --rm -it \
   -v $(pwd):/config \
   linuxserver/ffmpeg \
   -i "/config/$file_to_convert.mkv" \
   -c:v libx265 \
   -an \
   -x265-params crf=25 \
   "/config/$file_to_convert.mp4"

docker run --rm -it \
   -v $(pwd):/config \
   linuxserver/ffmpeg \
   -i "/config/$file_to_convert.mkv" \
   -c:v libx265 \
   -c:a copy \
   "/config/$file_to_convert.mp4"
```

- [Can I add youtube channels or content to my plex library?](https://www.reddit.com/r/PleX/comments/pxkx37/can_i_add_youtube_channels_or_content_to_my_plex/)
- [Naming and Organizing Your TV Show Files](https://support.plex.tv/articles/naming-and-organizing-your-tv-show-files/)
- [YouTube-Agent.bundle](https://github.com/ZeroQI/YouTube-Agent.bundle)
- [FFMPEG Container](https://hub.docker.com/r/linuxserver/ffmpeg)
- [FFMPEG - MKV to x265 MP4](https://superuser.com/questions/785528/how-to-generate-an-mp4-with-h-265-codec-using-ffmpeg)

## Maintain

Make a cron job that executes the build script.

## Manage

### Add new books in batch

Move all of your books over

```bash
mv /root/workspace/infra_containers/docker-media_backups/overdrive/ebooks/backups*.epub /storage2/books_import
```

```bash
docker-compose exec calibre-web bash
calibredb add --with-library /books /import/*
```

This will add everything as root, so I need to set ownership back as docker.

```bash
chown docker:docker -R /storage2/book_library/
```
