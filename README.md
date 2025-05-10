# logdy-docker

This is the Git repo of the docker [logdy](https://logdy.dev/) image. 

This image was created as alternative for [this feature request](https://github.com/logdyhq/logdy-core/issues/52). Logdy wasnt designed to be used as in a docker environment, but it does make it easier to deploy and run. Also it may be used in `kubernetes` clusers as sidecar container for applications on staging.

## What is logdy?

Logdy is a web-based platform designed to help monitor, track, and analyze application logs in real-time locally. It is a multi-purpose DevOps tool that enhances productivity in the terminal.

Logdy improves productivity by recording the output of the processes, whether that's standard output or a file, and routes it to a web UI. The web UI is served by Logdy on a specific port and accessible on a local host. The UI is a reactive, low-latency web application that automatically generates filters and allows you to browse and search through the logs.

## Quick reference

* Official github repo of logdy https://github.com/logdyhq/logdy-core
* Issues for this image https://github.com/rickraven/logdy-docker/issues
* Logdy documentation https://logdy.dev/docs/what-is-logdy

## How to use this image

Simple command:

```
docker run -d -p 8080:8080 -p 10800:10800 rickraven/logdy:0.16.0
```

Then you can visit `http://localhost:8080/` in your browser. By default logdy in this image works in [socket mode](https://logdy.dev/docs/explanation/command-modes#socket) so you mas send logs for 10800 port. Example of simple `fluent-bit` log pipeline with output to logdy:

```
pipeline:
  inputs:
    - name: kubernetes_events
      tag: k8s_events
      kube_url: https://kubernetes.default.svc
      tls.verify: on
  outputs:
    # Just send logs to logdy socket
    - name: tcp
      match: 'k8s_events'
      tls: off
      host: 127.0.0.1
      port: 10800
      format: json_lines
```

You may run container for log files following:

```
docker run -d -p 8080:8080 -v /var/logs:/data -e LOGDY_MODE="follow" -e LOGDY_FOLLOW_FILES="/data/*.log" rickraven/logdy:0.16.0
```

Example `docker-compose.yml` for `logdy`:

```
version: '3.1'

services:
  logdy:
    image: rickraven/logdy:0.16.0
    restart: always
    ports:
      - 8080:8080
      - 10800:10800
      - 10900:10900
      - 11000:11000
    environment:
      LOGDY_PASS: "mypassword"
      LOGDY_MODE: "socket"
      LOGDY_SOCKET_PORTS: "10800 10900 11000"
```

## Variables

* LOGDY_PORT - Port on which the Web UI will be served (default: 8080).
* LOGDY_PASS - Password for logdy Web interface.
* LOGDY_MAX_MESSAGE - Determines the maximum number of messages that will reside in the buffer. Buffer is an ordered FIFO queue with a fixed size (default: 1000).
* LOGDY_API_KEY - API key to be used when communicating with Logdy through the [API](https://logdy.dev/docs/reference/rest-api). If not set, it will be automatically generated and output to the startup log.
* LOGDY_MODE - Image may be work in `follow` and `socket` modes (default: socket).
* LOGDY_FOLLOW_FILES - Space separated list of files for following. Working only in `follow` mode.
* LOGDY_SOCKET_PORTS - Space separated list of ports which listens by logdy. Working only in `socket` mode.
