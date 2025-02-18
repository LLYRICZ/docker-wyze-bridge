# RTMP/RTSP/HLS Bridge for Wyze Cam

Quick docker container to enable RTMP, RTSP, and HLS streams for Wyze cams based on [noelhibbard's script](https://gist.github.com/noelhibbard/03703f551298c6460f2fd0bfdbc328bd#file-readme-md) with [kroo/wyzecam](https://github.com/kroo/wyzecam) and [aler9/rtsp-simple-server](https://github.com/aler9/rtsp-simple-server). 

Exposes a local RTMP, RTSP, and HLS stream for all your Wyze Cameras including v3. No Third-party or special firmware required.

Has been tested on MacOS, but should work on most x64 systems as well as on some arm-based systems like the raspberry pi. 
[See here](#armraspberry-pi-support) for instructions to run on arm.

## Changes in v0.3.0

- 🆕 Adjustable Bitrate and Resolution. [See here](#Bitrate-and-Resolution) 
- 🆕 Nearly cloudless by limiting the number of calls to the wyze servers by storing auth tokens and cameras locally. Should also help prevent http errors. Can be disabled with `- FRESH_DATA=True`

## Usage

git clone this repo, edit the docker-compose.yml with your wyze credentials, then run `docker-composer up`.

Once you're happy with your config you can use `docker-compose up -d` to run it in detached mode.


## URLs

`camera-nickname` is the name of the camera set in the Wyze app and are converted to lower case with hyphens in place of spaces. 

e.g. 'Front Door' would be `/front-door`


- RTMP:  
```
rtmp://localhost:1935/camera-nickname
```
- RTSP:  
```
rtsp://localhost:8554/camera-nickname
```
- HLS:  
```
http://localhost:8888/camera-nickname/stream.m3u8
```
- HLS can also be viewed in the browser using:
```
http://localhost:8888/camera-nickname
```


## Filtering

The default option will automatically create a stream for all the cameras on your account, but you can use the following environment options in your `docker-compose.yml` to filter the cameras.

All options are cAsE-InSensiTive, and take single or multiple comma separated values.


#### Examples:

- Whitelist by Camera Name (set in the wyze app):
```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_NAMES=Front Door, Driveway, porch
```
- Whitelist by Camera MAC Address:
```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_MACS=00:aA:22:33:44:55, Aa22334455bB
```
- Whitelist by Camera Model:
```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_MODEL=WYZEC1-JZ
```
- Whitelist by Camera Model Name:
```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_MODEL=V2, v3, Pan
```
- Blacklisting:

You can reverse any of these whitelists into blacklists by adding *block, blacklist, exclude, ignore, or reverse* to `FILTER_MODE`. 

```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_NAMES=Bedroom
    - FILTER_MODE=BLOCK
```

## Other Configurations

#### ARM/Raspberry Pi Support

The default configuration will use the x64 tutk library, however, you can edit your `docker-compose.yml` to use the 32-bit arm library by setting `dockerfile` as `Dockerfile.arm`:

```YAML
    wyzecam-bridge:
        container_name: wyze-bridge
        restart: always
        build: 
            context: ./app
            dockerfile: Dockerfile.arm
        environment:
            - WYZE_EMAIL=
            - WYZE_PASSWORD=
```

---
#### Bitrate and Resolution

Bitrate and resolution of the stream from the wyze camera can be adjusted with `- QUALITY=HD120`.
- Resolution can be set to `SD` (640x360 cams/480x640 doorbell) or `HD` (1920x1080 cam/1296x1728 doorbell). Default - HD.
- Bitrate can be set from 60 to 240 kb/s. Default - 120.
- Bitrate and resolution changes will apply to ALL cameras with the current version.

```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - QUALITY=SD60
```

---
#### rtsp-simple-server
[rtsp-simple-server](https://github.com/aler9/rtsp-simple-server) options can be configured by editing `/app/rtsp-simple-server.yml`.

In particular, increasing **readBufferCount** seems to help if you are getting dropped frames from your camera.

---
#### Custom FFmpeg Commands

You can pass a custom [command](https://ffmpeg.org/ffmpeg.html) to FFmpeg by using `FFMPEG_CMD` in your docker-compose.yml:

```YAML
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FFMPEG_CMD=-f h264 -i - -vcodec copy -f flv rtmp://rtsp-server:1935/
```
Additional info:
- The `ffmpeg` command is implied and is optional.
- The camera name will automatically be appended to the command, so you need to end with the rtmp/rtsp url.

---

## Debugging options

`- DEBUG_FFMPEG=True` Enable additional logging from FFmpeg

`- FRESH_DATA=True` Disable local cache and pull new data from wyze servers.

