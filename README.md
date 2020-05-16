# dwc2-klipper

## Credit

This is a branch from [seanauff dwc2-klipper repo](https://github.com/seanauff/dwc2-klipper) (credit to seanauff).

The change is for my personal use to support multipler printers from a single Raspberry Pi. 


## Context

dwc2-klipper is a [Docker image](https://hub.docker.com/repository/docker/stevenvo/dwc2-klipper) for running [DWC2] and [Klipper] for 3d Printing controllers. It is designed to run on Raspberry Pi or similar.

Referencs:

* https://klipper.info/klipper-+-dwc2-1/things-to-know-about-klipper-and-dwc2
* Because of recent changes to Klipper, this utilizes the fork from pluuuk as discussed [here](https://github.com/Stephan3/dwc2-for-klipper/issues/73).

## Pre-requisites
Ensure docker and docker-compose are installed on your host. Some references:

* [How to install Docker on Raspberry Pi](https://phoenixnap.com/kb/docker-on-raspberry-pi)
* [How to install Docker Compose on Ubuntu](https://phoenixnap.com/kb/install-docker-compose-ubuntu), best bet for Rpi: `sudo apt-get install docker-compose`


## Prepare your printer.cfg file

### Add DWC2 config

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

### Where to place your printer.cfg files

For each printer config file. place the `printer.cfg` file into a separate folder on your host (Raspberry Pi). You can rename the folder to your printer name, for example:

```
$ ls -lsh
4.0K drwxr-xr-x 2 pi pi 4.0K May 16 11:44 cr10s-klipper-printer.cfg
4.0K drwxr-xr-x 2 pi pi 4.0K May 16 11:34 ender3-klipper-printer.cfg
4.0K drwxr-xr-x 2 pi pi 4.0K May 16 11:35 kossel-klipper-printer.cfg
```

As above, `cr10s-klipper-printer.cfg` is a folder on host, it containers a `printer.cfg` file. This folder will be shared to the corresponding container. This is configured in `volume` section in `docker-compose.yaml`. Feel free to change the folder name, just needs to ensure same name is updated into `docker-compose.yaml`.


**Why do this?** This folder structure will resolve the `save_config` error due to Klipper unable to rename the `printer.cfg` to the backup config file (klipper [code](https://github.com/KevinOConnor/klipper/blob/master/klippy/configfile.py#L314)) due to lack of permission. 


## docker-compose.yaml
To add/edit/remove printers, update your `docker-compose.yaml` file using your text edit, notice there are 3 sections like below:

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

- service name (dwc2-klipper-cr10s): any name you like 
- `container_name`: any printer name you like
- `volumes`
	- path before `:`: the folder containing the `printer.cfg` file on your host.
	- path after `:`: the folder containing `printer.cfg` file on your container. 

## Spin up the container

On host (Raspberry Pi), execute `docker-compose up -d` 

To ssh into the container from host, execute `docker exec -it cr10s /bin/bash` (change cr10s to the container name defined in `docker-compose.yaml`)

