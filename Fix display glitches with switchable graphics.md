# Fix multi-monitor problems on laptops with Intel and NVIDIA GPUs

## OS and Hardware for testing:
* HP Omen an-113nm 2019.
* Intel UHD 630 and NVIDIA GeForce GTX 1050
* Three screens with connections: Type-C -> DP (Intel output), HDMI - HDMI (NVIDIA output), miniDP - DP (NVIDIA outpu)
* [Archlinux GUI](https://archlinuxgui.in/) OS
* [KDE Plasma](https://kde.org/) Desktop environment

## Preparations
1. Update [BIOS/UEFI](https://www.wikihow.com/Update-Your-Computer%27s-BIOS) to latest version
3. Update the OS:
```shell
sudo pacman -Syyu
```
4. Download all the dependencies for optimus-manager package:
```shell
sudo pacman -S mesa-utils python-dbus python-setuptools python xorg-xrandr git acpi_call bbswitch xf86-video-intel extra-cmake-modules nvidia qt5-base qt5-svg qt5-x11extras qt5-tools
```
## Installing optimus-manager
1. Clone optimus-manager git repo
```shell
git clone https://aur.archlinux.org/optimus-manager.git
```
2. Build the source
```shell
makepkg -si
```
4. Reboot the system
```shell
sudo reboot now
```
5. Check the status of systemd unit optimus-manager:
```shell
[lljubojevic@Omen17 ~]$ sudo systemctl status optimus-manager.service
● optimus-manager.service - Optimus Manager Commands Daemon
     Loaded: loaded (/usr/lib/systemd/system/optimus-manager.service; enabled; preset: disabled)
     Active: active (running) since Mon 2022-10-03 16:54:59 CEST; 39min ago
    Process: 434 ExecStartPre=/usr/bin/python3 -u -m optimus_manager.hooks.pre_daemon_start (code=exited, status=0/SUCCESS)
    Process: 505 ExecStartPre=/usr/bin/python3 -u -m optimus_manager.hooks.pre_xorg_start (code=exited, status=0/SUCCESS)
   Main PID: 891 (python3)
      Tasks: 1 (limit: 38278)
     Memory: 29.3M
        CPU: 3.668s
     CGroup: /system.slice/optimus-manager.service
             └─891 /usr/bin/python3 -u -m optimus_manager.daemon
```
6. **If your system used bumblebee for GPU switching, DISABLE IT**
```shell
sudo systemctl disable bumblebeed.service
```
## Editing optimus-manager config files:
1. In file /etc/default/grub load kernel parameter for hybrid screen management (both cards on):
```shell
GRUB_CMDLINE_LINUX="optimus-manager.startup=hybrid"
```
3. Generate new GRUB config:
```shell
sudo grub-mkconfig
```
5. Create optimus-manager config file /etc/optimus-manager/optimus-manager.conf as follows:
```shell
[optimus]
startup_mode=hybrid
startup_auto_battery_mode=hybrid
startup_auto_extpower_mode=hybrid
```
6. In SDDM config file /etc/sddm.conf place # characters in front of the lines starting with DisplayCommand and DisplayStopCommand
8. Disable KDE screen manager:
    * Settings - Startup - Background services - kscreen
9. Remove KDE screen manager:
```shell
sudo pacman -R kscreen
```
10. Use xrandr to find out your display connection names:
```shell
[lljubojevic@Omen17 ~]$ xrandr --listmonitors
Monitors: 3
 0: +*DP-1 1920/698x1080/393+1920+0  DP-1
 1: +DP-1-1 1920/598x1080/336+0+0  DP-1-1
 2: +HDMI-1-0 1920/527x1080/296+3840+0  HDMI-1-0
```
11. Create a BASH script in home directory /home/YOU/Displays.sh which will automatically set your prefered screen setup, use the following example and modify it:
```shell
#!/bin/bash
xrandr --output eDP-1 --auto --off
xrandr --output DP-1 --primary
xrandr --output DP-1-1 --auto --left-of DP-1
xrandr --output HDMI-1-0 --auto --right-of DP-1
```
12. Create a systemd unit on path /etc/systemd/user/display.service that will start the script with content:
```shell
[Unit]
Description=Set xrandr

[Service]
Type=simple
TimeoutStartSec=0
ExecStartPre=/bin/sleep 60
ExecStart=/home/lljubojevic/Display.sh

[Install]
WantedBy=default.target
```
13. Reload the process daemon
```shell
sudo systemctl daemon-reload
```
15. Enable unit start as a user:
```shell
sudo systemctl --user enable display.service
```
16. Reboot
```shell
sudo reboot now
```
