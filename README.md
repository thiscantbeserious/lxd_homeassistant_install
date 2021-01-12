# Home Assistant install with LXD (tested on Ubuntu Server)

The orginal script was aimed to be installed on Proxmox exlcusively - this was forked and modified for LXD on any Debian based OS. It is currently being used on an Ubuntu Server 20.10 aarch64 host on a RaspberryPi 4 8GB executing the current upstream supervisor install with all dependencies, also exposing the port (8123) on the host.

TLDR: This is not aimed just at Proxmox but a generic debian based install (Ubuntu Server still recommended, don't ask for support otherwise).

## Current state:

This is hardcoded to aarch64 for now. You need to modify the script (simply changing the machine-type) for other architectures. It currently completes successfully - select N upon install to keep network connectivity after reboot (overwriting will remove eth0 and hence the internet connectivity). 

However after stopping and restarting the LXC container some docker containers (DNS, Multicast) itself seem to exit with code 0, without any logs attached.

This has still to be solved. Maybe it's a better idea to rewrite HassOS Buildroot base to something like https://github.com/Linutronix/elbe to get a minimal Debian system as the base where you can properly install other packages in addition to Hass.io. LXD/LXC would be nice to have, especially for the snapshots (that are more reliable then those that are inbuild, especially since it stops/freezes the container before snapshotting and restarts it afterwards - so no Database corruption will happen) but it could be that Docker is just to much pain on LXC (I've already included the overlay kernel module for the container, maybe there are some others missing). Will have to verify ...

Edit: Or maybe this is just a simple restart policy issue. Maybe a systemd service would be better but I don't really have time to dive into this deeply right now ... so try running the following from within the 'homeassistant' container after the initial setup:

    docker update --restart unless-stopped $(docker ps -q)
    
----

Original Readme - see here: https://github.com/whiskerz007/proxmox_hassio_lxc/blob/master/README.md
