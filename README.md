# dwc2-klipper

## Credit

This is a branch from [seanauff dwc2-klipper repo](https://github.com/seanauff/dwc2-klipper) (credit to seanauff).

The change is for my personal use to support multipler printers from a single Raspberry Pi. 


## Context

dwc2-klipper is a [Docker image](https://hub.docker.com/repository/docker/stevenvo/dwc2-klipper) for running [DWC2] and [Klipper] for 3d Printing controllers. It is design to run on Raspberry Pi or similar.

Referencs:

* https://klipper.info/klipper-+-dwc2-1/things-to-know-about-klipper-and-dwc2
* Because of recent changes to Klipper, this utilizes the fork from pluuuk as discussed [here](https://github.com/Stephan3/dwc2-for-klipper/issues/73).


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

Place in a known place on your docker host (Raspberry Pi), which you will mount when starting the container using below instructions.

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
      - "/home/pi/dev/dwc2-klipper/klipper-printer-cfg-files/klipper-printer.cfg/cr10s_skr_1.3_bltouch-printer.cfg:/home/dwc2-klipper/config/printer.cfg"
    environment:
      TZ: America/Los_Angeles
    restart: unless-stopped
```
Each of that section is used for creating a docker container for each of your printer. 

Go ahead to update the 

- service name (dwc2-klipper-cr10s) to your own definition
- the `container_name` to your own definition
- For `volumes`, change the path before `:` to where the corresponding klipper printer.cfg file on your host (Raspberry Pi) locates. 

## Spin up the container

on host (Raspberry Pi), execute `docker-compose up -d` 

