manjaro-optimus-config
----------------------

This is a set of configuration files to dynamically enable/disable nVidia GPU in Optimus-enabled laptops without reboot (but with a display-manager restart, so you will have to login
again).

Tested and definitely working with Lenovo ThinkPad X1 Extreme and Linux kernel 5.0+.
Uses PCI power management facilities of newer kernels and does not mess with ACPI tables
(which makes the whole system much more stable).

I use lightdm as a display manager, but you can adjust scripts to work with any other
manager too.

The whole thing is, actually, pretty simple:

 1. We're reconfiguring Xorg server to use intel or nvidia specific configuration depending
    on the chosen mode (`diff nvidia/x11-optimus.conf intel/x11-optimus.conf`)
 2. Running a script before the display manager starts:
    - For the intel mode we're unloading nvidia kernel modules and disabling nvidia
      PCI devices (I have two, GPU and HDMI sound). You can find device ids by running
      `lspci | grep -i nvidia`
    - For the nvidia mode we're rescanning the PCI bus (for nvidia devices to reappear)
      and loading nvidia-specific kernel modules
 3. Adjusting the lightdm screen configuration screen to run `xrandr` with
    mode-specific parameters
 4. Restart the display manager

Installation
------------

I'm using `lightdm`, you might have to tweak the installation scripts to
work with other display managers.

1. Clone the repo
2. If you used `mhwd` to install video drivers, `mhwd` will create few
   configuration files under `/etc/X11/xorg.conf.d/`, `/etc/modules-load.d/` and
   `/etc/modprobe.d/`. Either remove them manually or just uninstall drivers through
   `mhwd` to clean things up.
3. Uninstall `bumblebee`, if you have it installed
3. Install nvidia driver: `pacman -S linux50-nvidia`
4. Install intel driver: `pacman -S xf86-video-intel`
5. Adjust packaged Xorg config files to your liking (`intel/x11-optimus.conf` and
   `nvidia/x11-optimus.conf`). You might want to remove everything but video-related
   sections. 
6. Adjust intel or nvidia related kernel module options in `shared/modprobe-optimus-gpu.conf`
7. Install an `optimus.service` by running:
   `cp shared/optimus.service /etc/systemd/system/`
   `systemctl enable optimus.service`
8. Run `sudo ./set-mode intel`. This will restart your display service and disable
   nVidia GPU. It should work out of the box. On the next reboot it'll also apply
   various module options (power saving for nvidia, etc) automatically.

Whenever you want to switch between intel or nvidia GPUs, just run `sudo ./set-mode <mode>`
where `<mode>` is either intel or nvidia.

