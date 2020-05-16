# dwc2-klipper

## Credit

This is a branch from [seanauff dwc2-klipper repo](https://github.com/seanauff/dwc2-klipper) (credit to seanauff).

The change is for my personal use to support multipler printers from a single Raspberry Pi. 


## Context

dwc2-klipper is a [Docker image](https://hub.docker.com/repository/docker/stevenvo/dwc2-klipper) for running [DWC2] and [Klipper] for 3d Printing controllers. It is design to run on Raspberry Pi or similar.

Referencs:

* https://klipper.info/klipper-+-dwc2-1/things-to-know-about-klipper-and-dwc2
* Because of recent changes to Klipper, this utilizes the fork from pluuuk as discussed [here](https://github.com/Stephan3/dwc2-for-klipper/issues/73).

## Pre-requisites
Ensure docker and docker-compose are installed on your host. Some references:

* [How to install Docker on Raspberry Pi](https://phoenixnap.com/kb/docker-on-raspberry-pi)
* [How to install Docker Compose on Ubuntu](https://phoenixnap.com/kb/install-docker-compose-ubuntu), best bet for Rpi: `sudo apt-get install docker-compose`


## Prepare your printer.cfg file

Assuming you already have a klipper printer.cfg file, add these lines into each of your printer.cfg files.

```cfg
[virtual_sdcard]
path: /home/dwc2-klipper/sdcard

[web_dwc2]
## optional - defaulting to Klipper
printer_name: Klipper
# optional - defaulting to 0.0.0.0
listen_adress: 0.0.0.0
# needed - use above 1024 as nonroot
listen_port: 4750
# optional defaulting to dwc2/web. Its a folder relative to your virtual sdcard.
web_path: dwc2/web
```

### Structure your printer.cfg files

Place each `printer.cfg` file into a separate folder. The `docker-compose.yaml` shares each config folder on host with the corresponding containers. Example structure used by the docker-compose currently:

```
$ ls -lsh
4.0K drwxr-xr-x 2 pi pi 4.0K May 16 11:44 cr10s-klipper-printer.cfg
4.0K drwxr-xr-x 2 pi pi 4.0K May 16 11:34 ender3-klipper-printer.cfg
4.0K drwxr-xr-x 2 pi pi 4.0K May 16 11:35 kossel-klipper-printer.cfg
```
`cr10s-klipper-printer.cfg` is a folder on host, containers a `printer.cfg` file and will be shared to each container. Feel free to change this folder name, but ensure to update `docker-compose.yaml` with the new name in `volume` section.

```
$ ls -lsh cr10s-klipper-printer.cfg
8.0K -rw-r--r-- 1 pi   pi   4.2K May 16 11:28 printer-20200516_184449.cfg
8.0K -rw-r--r-- 1 root root 4.1K May 16 11:44 printer.cfg
```

This structure will resolve the `save_config` error due to the container fails to [rename the printer.cfg to backup config](https://github.com/KevinOConnor/klipper/blob/master/klippy/configfile.py#L314). 


## Update docker-compose.yaml file
Open `docker-compose.yaml` file using your text edit, notice there are 3 sections like below:

```
  dwc2-klipper-cr10s:
    image: "stevenvo/dwc2-klipper:arm"
    container_name: cr10s
    privileged: true
    ports:
      - "4750:4750"
    volumes:
      - "/home/pi/cr10s-klipper-printer.cfg:/home/dwc2-klipper/config"
    environment:
      TZ: America/Los_Angeles
    restart: unless-stopped
```
Each of that section is used for creating a docker container for each of your printer. 

Go ahead to update the 

- service name (dwc2-klipper-cr10s) to your own definition
- the `container_name` to your own definition
- For `volumes`, change the path before `:` to where the corresponding foldet containing the `printer.cfg` file on your host (Raspberry Pi) locates. 

## Spin up the container

On host (Raspberry Pi), execute `docker-compose up -d` 

To ssh into the container from host, execute `docker exec -it cr10s /bin/bash` (change cr10s to the container name defined in `docker-compose.yaml`)

