This Image is built and used by [prind](.).

# Moonraker packaged in Docker
## What is Moonraker?

>Moonraker is a Python 3 based web server that exposes APIs with which client applications may use to interact with the 3D printing firmware Klipper. Communcation between the Klippy host and Moonraker is done over a Unix Domain Socket. Tornado is used to provide Moonraker's server functionality.

_via https://moonraker.readthedocs.io/en/latest/_

## Usage
Moonraker requires Klipper to operate. See the Images' [README.md](../klipper/README.md) on how to run the Klipper Image.  
Both Containers need to share their `run` Volume in order to be able to communicate via klippers unix socket. Configure Moonraker to use `/opt/printer_data/run/klipper.sock` as klippys uds address.  

Create `moonraker.conf` and `printer.cfg` as well as the directories `run` and `gcode`, then run the containers.

#### Run
```bash
docker run \
  --privileged \
  -v /dev:/dev \
  -v $(pwd)/run:/opt/printer_data/run \
  -v $(pwd)/gcode:/opt/printer_data/gcodes \
  -v $(pwd)/printer.cfg:/opt/printer_data/config/printer.cfg \
  mkuf/klipper:latest

docker run \
  -v $(pwd)/run:/opt/printer_data/run \
  -v $(pwd)/gcode:/opt/printer_data/gcodes \
  -v $(pwd)/moonraker.conf:/opt/printer_data/config/moonraker.conf \
  -p 7125:7125 \
  mkuf/moonraker:latest
```

#### Compose
```yaml
services:
  klipper:
    image: mkuf/klipper:latest
    privileged: true
    volumes:
      - /dev:/dev
      - ./printer.cfg:/opt/printer_data/conf/printer.cfg
      - ./run:/opt/printer_data/run
      - ./gcode:/opt/printer_data/gcodes

  moonraker:
    image: mkuf/moonraker:latest
    ports:
      - "7125:7125"
    volumes:
      - ./moonraker.conf:/opt/printer_data/conf/moonraker.conf
      - ./run:/opt/printer_data/run
      - ./gcode:/opt/printer_data/gcodes
```

## Defaults
|Entity|Description|
|---|---|
|User| `moonraker (1000:1000)` |
|Workdir|`/opt`|
|Entrypoint|`/opt/venv/bin/python moonraker/moonraker/moonraker.py`|
|Cmd|`-d /opt/printer_data/`|

## Ports
|Port|Description|
|---|---|
|`7125/tcp`|Default Webapi Port|

## Volumes
|Volume|Description|
|---|---|
|`/opt/printer_data/run`| Default Location for runtime Files generated by Klipper. Used to access `klipper.sock`, the unix socket that is used for communation with klipper |
|`/opt/printer_data/config`|Config directory to host `moonraker.conf`|
|`/opt/printer_data/gcodes`|Stores uploaded GCODE Files|
|`/opt/printer_data/logs`|Log directory for Klipper and Moonraker|

## Tags
|Tag|Description|Static|
|---|---|---|
|`latest`/`nightly`|Refers to the most recent runtime Image.|May point to a new build within 24h, depending on code changes in the upstream repository.|
|`<7-digit-sha>` <br>eg: `d37f91c`|Refers to a specific commit SHA in the upstream repository. eg: [Arksine/moonraker:d37f91c](https://github.com/Arksine/moonraker/commit/d37f91c9c864302e750385297d2aa2a0c9b43035)|Yes|

## Targets
|Target|Description|Pushed|
|---|---|---|
|`build`|Pull Upstream Codebase and build python venv|No|
|`run`|Default runtime Image|Yes|