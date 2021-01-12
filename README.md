## Intend of fork / explanation:

This was forked and modified for LXD running on Ubuntu Server 20.10 aarch64 on a RaspberryPi 4 8GB executing the current upstream supervisor install with all dependencies, also exposing the port (8123) on the host. So basically a complete install with any system that runs LXD (tested with the snapstore install).

This is not aimed specifically at Proxmox but a generic install.

## Current state:

This is hardcoded to aarch64 for now. You need to modify the script (simply changing the machine-type) for other architectures. It currently completes successfully - select N upon install to keep network connectivity after reboot (overwriting will remove eth0 and hence the internet connectivity). 

However after stopping and restarting the LXC container some docker containers (DNS, Multicast) itself seem to exit with code 0, without any logs attached.

This has still to be solved. Maybe it's a better idea to rewrite HassOS Buildroot base to something like https://github.com/Linutronix/elbe to get a minimal Debian system as the base where you can properly install other packages in addition to Hass.io. LXD/LXC would be nice to have, especially for the snapshots (that are more reliable then those that are inbuild, especially since it stops/freezes the container before snapshotting and restarts it afterwards - so no Database corruption will happen) but it could be that Docker is just to much pain on LXC (I've already included the overlay kernel module for the container, maybe there are some others missing). Will have to verify ...

Edit: Or maybe this is just a simple restart policy issue. Try running the following from within the 'homeassistant' container:

    docker update --restart unless-stopped $(docker ps -q)

----

Original Readme:

# Home Assistant in LXC container (managed by LXD)

Many benefits can be gained by using a LXC container compared to a VM. The resources needed to run a LXC container are less than running a VM. Modifing the resouces assigned to the LXC container can be done without having to reboot the container. The serial devices connected to Proxmox can be shared with multiple LXC containers simulatenously.

## Usage

***Note:*** _Before using this repo, make sure Proxmox is up to date._

To create a new LXC container on Proxmox and setup Home Assistant to run inside of it, run the following in a SSH connection or the Proxmox web shell.

```
bash -c "$(wget -qLO - https://github.com/whiskerz007/proxmox_hassio_lxc/raw/master/create_container.sh)"
```

After running the above command, you should modify the container's `Resources` (`Cores`, `Memory`, `Root disk`) before you continue to setup Home Assistant. Failure to do so could result in Home Assistant not functioning as expected. Modified `Resources` will be applied immediately without the need to reboot the container. Hardware changes to Proxmox (for example plugin/unplug USB device) requires the container to restart in order for Home Assistant to parse the changes.

## Update device hooks

To update the list of devices that are shared with the LXC ID of `100`, run the following in a SSH connection or the Proxmox web shell.

```
bash -c "$(wget -qLO - https://github.com/whiskerz007/proxmox_hassio_lxc/raw/master/set_autodev_hook.sh)" -s 100
```

***Note:*** _The changes will apply on the next start of LXC._

## Copy data between containers

To ease the process of updating the LXC configuration, a script has been provided. To copy Home Assistant data from one container to another, run the following in a SSH connection or the Proxmox web shell.

```
bash -c "$(wget -qLO - https://github.com/whiskerz007/proxmox_hassio_lxc/raw/master/copy_data.sh)"
```

## Known limitations

- Unable to use bluetooth devices due to the limitation of LXC
- Setting up container on a ZFS pool will cause issues with addons that use mySQL/MariaDB due to ZFS not implementing `fallocate` properly
- WireGuard addon might generate a warning for `IP forwarding` being disabled. To enable this feature you'll need to add `post-up echo 1 > /proc/sys/net/ipv4/ip_forward` to `/etc/network/interfaces` under Proxmox, then reboot Proxmox. [[Link](https://pve.proxmox.com/wiki/Network_Configuration#_masquerading_nat_with_tt_span_class_monospaced_iptables_span_tt)] 

## Console

There is no login required to access the console from the Proxmox web UI. If you are presented with a blank screen, press `CTRL + C` to generate a prompt.
