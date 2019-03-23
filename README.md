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
    on the chosed mode (`diff nvidia/x11-optimus.conf intel/x11-optimus.conf`)
 2. Updating the script that's executed before display manager starts:
    - For intel mode we're unloading nvidia kernel modules and disabling the nvidia
      PCI devices (I have two, GPU and HDMI sound). You can find device ids by running
      `lspci | grep -i nvidia`
    - For nvidia mode we're rescanning the PCI bus (for nvidia devices to reappear)
      and loading nvidia-specific kernel modules
 3. Adjusting the lightdm screen configuration screen to run `xrandr` with
    different parameters
 4. Restarting the display manager

Installation
------------

I'm using `lightdm`, you might have to tweak the installation scripts to
work with other display managers. See how it works section for more information.

1. Clone the repo
2. If you used `mhwd` to install video drivers, `mhwd` will create few
   configuration files under `/etc/X11/xorg.conf.d/`, `/etc/modules-load.d/` and
   `/etc/modprobe.d/`. Either remove them manually or just uninstall `mhwd` drivers
3. Uninstall bumblebee if you have it installed
3. Install nvidia driver: `pacman -S linux50-nvidia`
4. Install intel driver: `pacman -S xf86-video-intel`
5. Adjust packaged Xorg config files to your liking (`intel/x11-optimus.conf` and
   `nvidia/x11-optimus.conf`). You might want to remove everything but video-related
   stuff. 
6. Adjust intel or nvidia related kernel module options in `shared/modprobe-optimus-gpu.conf`
7. Update display-manager configuration file to run a script that will enable or
   disable nvidia PCI device before the display manager starts. I'm not a systemd
   guru, maybe there's a better way to do it. The goal here is to load necessary
   nvidia modules before X11 is started.

   1. Edit /etc/systemd/system/display-manager.service
   2. Add `ExecPreStart=/etc/X11/optimus-startup` after `[Service]`
   3. Save
   4. Run `sudo systemctl daemon-reload` to update the service
8. Run `sudo ./set-mode intel`. This will restart your display service and disable
   nVidia GPU. It should work out of the box. On the next reboot it'll also apply
   various module options (power saving for nvidia, etc) automatically.

Whenever you want to switch between intel or nvidia GPUs, just run `sudo ./set-mode <mode>`
where <mode> is either intel or nvidia.

