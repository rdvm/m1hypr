# m1hypr
A basic collection of resources for getting Hyprland running from Asahi Minimal Install

## MBA Minimal Asahi + Hyprland setup

If you've already installed Asahi and want to start from scratch, continue
here. Otherwise jump to [Asahi Minimal Install.](#asahi-minimal-install)

### Uninstalling Asahi

Following the example of [Mr. Macintosh's YouTube
Tutorial](https://youtube.com/watch?v=nMnWTq2H-N0)

* Open **Disk Utility**
  * **View** > **Show all devices**
  * Select top-level SSD and then **Partitions** to view all partitions
* Terminal `diskutil list`
  * Look for Asahi partitions and specifically EFI filesystem e.g. "EFI EFI -
    ASAHI"
  * Delete EFI volume with `diskutil eraseVolume JHFS+ drive /dev/diskXyX`
    * In my case and in multiple tutorials I've seen that device is
    `/dev/disk0s4` but could be different
* Go back to **Disk Utility**
  * Select the top-level SSD and then **Partition**
  * Remove the 3 partitions for Asahi, leaving the "Macintosh HD" and also first
  and last partitions on the disk
  * After applying the changes reboot
* You'll likely see a Recovery Assistant message like "Custom kernel failed to
  boot"
  * Select "Startup Disk" and select "Macintosh HD"

### Asahi Minimal Install

Check out the Asahi project for latest guidance but for today:

* `curl https://alx.sh | sh`
* `N` on "Expert mode"
* `r: Resize an existing partition to make space for a new OS`
* Pick how much space to leave for macOS (did 40% for macOS today)
* `f: Install an OS into free space`
* `2: Asahi Linux Minimal (Arch Linux ARM)`
* Pick how much space on new partition for Linux (default is max; I did max)
* Name for OS (default is "Asahi Linux"; I did "Asahi Linux")
* Follow instructions on screen to reboot in recovery and select the right OS
  to load etc.
* Follow the instructions to finish dual boot setup and reboot into Asahi

### Configure Arch/Hyprland

* At **Alarm login** enter `root` for username and password
* Configure networking by creating the file `/etc/iwd/main.conf` according to
  Section 4.3 of the Arch Wiki page for `iwd` (may need to create the directory
in `/etc`)

```
[General]
EnableNetworkConfiguration=true

[Network]
NameResolvingService=systemd
```

* Save and exit
* Enable and start `iwd`
  * `systemctl enable iwd`
  * `systemctl start iwd`

At this point you should have active networking with a wired connection. If
wired check for IP address (`ip a`), try `ping` on something to test. If you
need to configure WiFi: 

* Run `iwctl`
* Inside iwd:
* `device list`
* `station wlan0 connect <your SSID/wireless network name here>`
  * Enter the password for your wireless network when prompted.
* `exit`

Check for IP address and connectivity.

#### Update Arch & create user

* Update `/etc/pacman.d/mirrorlist` to comment out global server and uncomment
  a local server. In my experience repository actions were, not surprisingly,
way faster when I switched to a US mirror.

* Set locale by uncommenting correct line in `/etc/locale.gen` and placing that
  same value as the top/only line in `/etc/locale.conf` and running
`locale-gen`.
  * In my case that was `LANG=en_US.UTF-8`
* Set your timezone using `timedatectl`. Check out out the [Arch
  Wiki](https://wiki.archlinux.org/title/System_time#Time_zone) or other parts
of the internet for more info.

```
timedatectl set-timezone America/Chicago
```

* Update the system with 

```
pacman -Syu
```

* Create a new user account for yourself and add it to the `wheel` group.
  Replace `<username>` with your desired username

```
useradd -m -G wheel -s /bin/bash <username>
```

* Install `sudo` so your new user can become root as needed 

```
pacman -S sudo
```
* Add your user to the sudoers group. One way to do that is to uncomment an
  existing line in `/etc/sudoers` that grants root privileges to members of the
`wheel` group. If you're familiar with Vim, just run `visudo`, and it'll open
the file for editing. Otherwise you can use Nano or some other editor to open
and edit the file
  * Navigate to the line `# %wheel ALL=(ALL) ALL`
  * Uncomment the line by removing the `#` so it's just `%wheel ALL=(ALL) ALL`
  * Write and quit

* Create a password for your new user account

```
passwd <username>
```

* Reboot and sign in with new user

Next get NetworkManager installed, enabled, and use it to connect to WiFi
since it'll be a better experience for normal use.

* Install NetworkManager 

```
sudo pacman -S networkmanager
```

* Enable NetworkManager 

```
sudo systemctl enable NetworkManager.service
```

* Reboot (not strictly necessary but it's quick enough that you might as well)

* Connect to WiFi using `nmcli`. Replace `<SSID>` with the name of your
  wireless network and `<password>` with your WiFi password.

```
nmcli device wifi connect <SSID> password <password>
```

Check that you're getting a good IP address with `ip a`, able to ping other IP
addresses and/or hostnames with `ping 8.8.8.8`/`ping google.com`, etc.

#### Move to edge kernel

If you don't want to use the Asahi Edge kernel then skip this section, but if
you're doing something like Hyprland right now, you probably will want to be on
the edge kernel. You can read more on the [Asahi
blog.](https://asahilinux.org/2022/12/gpu-drivers-now-in-asahi-linux/)

```
sudo pacman -Syu
sudo pacman -S linux-asahi-edge mesa-asahi-edge
sudo update-grub
```

After those steps, reboot to load the new kernel.

#### WM and stuff

We're going to need an AUR helper so install Yay:

```
$ pacman -S --needed git base-devel
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ makepkg -si
```

##### Install Hyprland and other packages

You can install all the packages you'll need plus a few more by using the
`baseApps.txt` file in this repo. This is a collection of packages I installed
during setup, and in the time shortly thereafter as I realized what I was
missing. You can just copy and paste the contents of that file or clone the
repo, edit it to add/remove packages, and then run the command below to install
everything in one shot with `m1hypr/baseApps.txt` being whatever the path is
to the file on your system.

```
yay -S --needed - < m1hypr/baseApps.txt
```

At a minimum you'll need to install `hyprland-legacyrender` from the AUR since
that's the version of Hyprland required for these machines. And here are a few
more packages you'll very likely want, but this list is not exhaustive and none
of these are strictly necessary.

  * `sddm`
  * `polkit-gnome` or `polkit-kde-agent`
  * `xdg-desktop-portal-hyprland`
  * `swayidle`
  * `swaylock` or `swaylock-effects-git` if you want something a little fancier
  * `wofi` (This is an application launcher/run menu, and even if you
    ultimately decide to go with something else you might want it to get off
the ground, as you'll see later.)
  * `alacritty` (The default terminal for Hyprland is Kitty, and it's currently
    broken on Asahi Edge so you'll want some other terminal emulator.)

In order to use SDDM to login and start Hyprland, you'll need to enable it.

```
sudo systemctl enable sddm
```

Now you can reboot, and you should be greeted by the SDDM login screen. When
you sign in you'll see a notification at the top of your screen to let you know
you're using an autogenerated Hyprland config and there will be a couple basic
keybindings displayed as well. Unfortunately, with the default terminal being
Kitty, the binding to open a terminal won't work.

In the default config `mod + r` will open a Wofi application launcher/run menu,
so you can use that to launch whatever terminal emulator you installed so that
you can open up the file `~/.config/hypr/hyprland.conf` to begin editing,
starting, perhaps, by replacing `kitty` with your terminal.
