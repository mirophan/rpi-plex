# Official SMA container based on linuxserver/radarr
Docker container for Radarr that includes all FFMPEG and python requirements to run SMA with Radarr.

## Version Tags

|Tag|Description|
|---|---|
|latest|Stable release from Radarr with precompiled FFMPEG binaries|
|build|Stable release from Radarr with FFMPEG compiled from jrottenberg/ffmpeg|
|nightly|Nightly release from Radarr with precompiled FFMPEG binaries|

## Usage

### Recent update
As of 3/9/2020 the containers were overhauled and the location of the script was changed from `/usr/local/bin/sma/sickbeard_mp4_automator` to `/usr/local/sma`. The autoProcess mount point has been modified as well to be more docker friendly in a `/usr/local/sma/config` directory. Please review and update accordingly.

### docker-compose
~~~
services:
  radarr:
    image: mdhiggins/radarr-sma
    container_name: radarr
    volumes:
      - /opt/appdata/radarr:/config
      - /opt/appdata/sma:/usr/local/sma/config
      - /mnt/storage/movies:/movies
      - /mnt/storage/downloads:/downloads
    ports:
      - 7878:7878
    restart: always
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
~~~

### autoProcess.ini
- Mount autoProcess.ini containing directory to `/usr/local/sma/config` using volumes
 - Consider making this writable as new options will be auto written to the config as they are added
- Radarr configuration options are read from `config.xml` inside the container and injected at runtime into `autoProcess.ini`
 - ffmpeg
 - ffprobe
 - host (read from environment variable or set to 127.0.0.1)
 - webroot
 - port
 - ssl

### FFMPEG Binaries
- `/usr/local/bin/ffmpeg`
- `/usr/local/bin/ffprobe`

## Configuring Radarr

###  Enable completed download handling
- Settings > Download Client > Completed Download Handling > Enable: Yes

### Add Custom Script
- Settings > Connect > + Add > Custom Script

|Parameter|Value|
|---|---|
|On Grab| No|
|On Download| Yes|
|On Upgrade| Yes|
|On Rename| No|
|Path|`/usr/local/sma/postRadarr.sh`|
|Arg||

## Logs

Located at `/usr/local/sma/config/sma.log` inside the container and your mounted config folder

## Environment Variables

|Variable|Description|
|---|---|
|PUID|User ID|
|PGID|Group ID|
|HOST|Local IP address for callback requests, default `127.0.0.1`|
|SMA_PATH|`/usr/local/sma`|
|SMA_UPDATE|Default `false`. Set `true` to pull git update of SMA on restart|

## Special Considerations
Using the `build` tag leverages mulit-stage docker builds to generate FFMPEG compiled using [jrottenberg/ffmpeg's](https://hub.docker.com/r/jrottenberg/ffmpeg) containers. This allows flexibility with building FFMPEG using special options such as VAAPI or NVENC. Building locally allows `ARG` values to be set to change the underlying parent container tags as below

|ARG|Default|Description|
|---|---|---|
|ffmpeg_tag|latest|Set tag to correspond to jrottenberg/ffmpeg:tag|
|radarr_tag|latest|Set tag to correspond to linuxserver/radarr:tag|

### VAAPI docker-compose sample
~~~
services:
  radarr:
    container_name: radarr
    build:
      context: https://github.com/mdhiggins/radarr-sma.git#build
      args:
        - ffmpeg_tag=4.2-vaapi
~~~
