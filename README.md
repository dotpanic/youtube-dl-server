![Docker Stars Shield](https://img.shields.io/docker/stars/dotpanic/youtube-dlc-server.svg?style=flat-square)
![Docker Pulls Shield](https://img.shields.io/docker/pulls/dotpanic/youtube-dlc-server.svg?style=flat-square)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://raw.githubusercontent.com/dotpanic/youtube-dlc-server/master/LICENSE)

# youtube-dlc-server

Simple Web and REST interface for downloading youtube videos onto a server. [`bottle`](https://github.com/bottlepy/bottle) + [`youtube-dlc`](https://github.com/blackjack4494/youtube-dlc).

Forked from [nbr23/youtube-dl-server](https://github.com/nbr23/youtube-dl-server).

![screenshot][1]

![screenshot][2]

## Running

For easier deployment, a docker image is available on [dockerhub](https://hub.docker.com/r/dotpanic/youtube-dlc-server).

### Docker CLI

This example uses the docker run command to create the container to run the app. Note the `-v` argument to specify the volume and its binding on the host. This directory will be used to output the resulting videos.

```shell
docker run -d --name youtube-dlc -v $HOME/youtube-dlc:/youtube-dlc dotpanic/youtube-dlc-server -p 8080:8080
```

### Docker Compose

This is an example service definition that could be put in `docker-compose.yml`.

```yml
  youtube-dlc:
    image: "dotpanic/youtube-dlc-server"
    volumes:
      - $HOME/youtube-dlc:/youtube-dlc
    ports:
      - "8080"
    restart: always
```

#### Configuration
For easier configuration management and edition, you can save your variables in an external file and source them in your docker-compose.yml like the following example.

Configuration file `config.env`:

```
YDL_RECODE_VIDEO_FORMAT="webm"
YDL_SUBTITLES_LANGUAGES="en"
```

docker-compose.yml:
```yml
  youtube-dlc:
    image: "dotpanic/youtube-dlc-server"
    env_file:
      - ./config.env
    volumes:
      - $HOME/youtube-dlc:/youtube-dlc
    ports:
      - "8080"
    restart: always
```

### Python

If you have python ^3.3.0 installed in your PATH you can simply run like this, providing optional environment variable overrides inline.

```shell
YDL_SERVER_PORT=8123 python3 -u ./youtube-dlc-server.py
```

## Usage

### Start a download remotely

Downloads can be triggered by supplying the `{{url}}` of the requested video through the Web UI or through the REST interface via curl, etc.

#### HTML

Just navigate to `http://{{host}}:8080/` and enter the requested `{{url}}`.

#### Curl

```shell
curl -X POST --data-urlencode "url={{url}}" http://{{host}}:8080/api/downloads
```

#### Fetch

```javascript
fetch(`http://${host}:8080/api/downloads`, {
  method: "POST",
  body: new URLSearchParams({
    url: url,
    format: "bestvideo"
  }),
});
```

#### Bookmarklet

Add the following bookmarklet to your bookmark bar so you can conviently send the current page url to your youtube-dlc-server instance.

##### HTTPS
If your youtube-dlc-server is served through https (behind a reverse proxy handling https for example), you can use the following bookmarklet:

```javascript
javascript:fetch("https://${host}/api/downloads",{body:new URLSearchParams({url:window.location.href,format:"bestvideo"}),method:"POST"});
```

##### Plain text
If you are hosting it without HTTPS, the previous bookmarklet will likely be blocked by your browser (as it will generate mixed content when used on HTTPS sites).
Instead, you can use the following bookmarklet:

```javascript
javascript:(function(){document.body.innerHTML += '<form name="ydl_form" method="POST" action="http://${host}/api/downloads"><input name="url" type="url" value="'+window.location.href+'"/></form>';document.ydl_form.submit()})();
```

## Implementation

The server uses [`bottle`](https://github.com/bottlepy/bottle) for the web framework and [`youtube-dlc`](https://github.com/blackjack4494/youtube-dlc) to handle the downloading. The integration with youtube-dlc makes use of their [python api](https://github.com/blackjack4494/youtube-dlc).

This docker image is based on [`python:alpine`](https://registry.hub.docker.com/_/python/) and consequently [`alpine:3.8`](https://hub.docker.com/_/alpine/).

[1]:youtube-dlc-server.png
[2]:youtube-dlc-server-logs.png
