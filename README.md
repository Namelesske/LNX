# LNX
Linux is Not Unix móktár

#Linux commands and notes
https://linuxisnotunix.blogspot.com/



#debian package source
Downgrade the current kernel to the kernel available in stable
Running sudo apt edit-sources
 
Add the stable repository:
 
deb http://http.debian.net/debian/ stable main contrib non-free
Only apt pin the kernel to stable
To prevent other packages downgrading to stable, we will only pin the kernel to the stable branch.
 
Create a new file /etc/apt/preferences.d/kernel with:
 
Package: linux-image-amd64
Pin: release a=stable
Pin-Priority: 950
 
Package: linux-headers-amd64
Pin: release a=stable
Pin-Priority: 950
 
and /etc/apt/preferences.d/apt-branches with:
 
Package: *
Pin: release a=testing
Pin-Priority: 650
 
Package: *
Pin: release a=stable
Pin-Priority: 600
For Debian to recognize the repositories, we will need to refresh the package list:
 
sudo apt update
Once this is done, the kernel and headers in stable can be installed:
 
sudo apt install linux-image-amd64/stable linux-headers-amd64/stable
 
#debian_extras
sudo apt install ttf-mscorefonts-installer rar unrar libavcodec-extra gstreamer1.0-libav gstreamer1.0-plugins-ugly gstreamer1.0-vaapi
 
#pacman commands cheat sheet
pacman -Syu <pkg> Install (and update package list)
pacman -S <pkg> Install only pacman -Rsc <pkg> Uninstall 
pacman -Ss <keywords> Search pacman -Syu Upgrade everything
pacman -Qe List explictly-installed packages 
pacman -Ql <pkg> What files does this package have? 
pacman -Qii <pkg> List information on package 
pacman -Qo <file> Who owns this file? 
pacman -Qs <query> Search installed packages for keywords
pacman -Qdt List unneeded packages 
pacman -Rns $(pacman -Qdtq) Uninstall unneeded packages
 
#prime
I have been plowing the internet for days over this issue. Had similar setup and problems. I chanced upon here about using DRI_PRIME=1 before any commands to use your dedicated GPU. So I tried
export $DRI_PRIME=1
 
and this worked. So I added a line it in /etc/environment, DRI_PRIME=1 , rebooted and now System Details shows my AMD card as default.
Caveat: I think this disables the Integrated Graphics and changes your system to just make use your dedicated GPU only
 
#debian contrib nonfree repos
As root you need to edit
sudo nano /etc/apt/sources.list
Then add contrib and non-free at the end of each line that begins with deb and deb-src just like the example:
deb http://http.us.debian.org/debian RELEASE main contrib non-free
deb http://security.debian.org RELEASE/updates main contrib non-free
Save the file, and run ‘apt-get update‘ and optionally ‘apt-get upgrade‘ to activate the changes.
#xanmod custom kernel DEIBAN & DEBIAN BASED
echo 'deb http://deb.xanmod.org releases main' | sudo tee /etc/apt/sources.list.d/xanmod-kernel.list && wget -qO - https://dl.xanmod.org/gpg.key | sudo apt-key add -
sudo apt update && sudo apt install linux-xanmod
#swapcheck
On Ubuntu 18.04 you can uninstall btrfs-support with
#TIMESHIFT DEPENDS DONT RECOMMEND IT
apt purge btrfs-progs
But that probably wouldn't save you much boot time. On my system the reason was, that I don't have a swap partition but on boot it is searched for such for about 30 seconds (while displaying the btrfs-scan).
#remove the swap check
open /etc/initramfs-tools/conf.d/resume
replace RESUME=UUID=xxx with RESUME=none
issue sudo update-initramfs -u
reboot your system
source: https://askubuntu.com/a/1034952/34298


#wait online

Use

systemctl disable systemd-networkd-wait-online.service


to disable the wait-online service to prevent the system from waiting on a network connection, and use

systemctl mask systemd-networkd-wait-online.service


to prevent the service from starting if requested by another service (the service is symlinked to /dev/null).


#silentboot

You would need to edit the file /etc/default/grub. In this file you'll find an entry called GRUB_CMDLINE_LINUX_DEFAULT. This entry must be edited to control the display of the splash screen.

The presence of the word splash in this entry enables the splash screen, with condensed text output. Adding quiet as well, results in just the splash screen; which is the default for the desktop edition since 10.04 (Lucid Lynx). In order to enable the "normal" text start up, you would remove both of these.

So, the default for the desktop, (i.e. splash screen only):

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash" #Hide text and show splash
For the traditional, text display:

GRUB_CMDLINE_LINUX_DEFAULT=        #Show text but not the splash
For the splash, but the ability to show the boot messages by pressing Esc:

GRUB_CMDLINE_LINUX_DEFAULT="splash"
Or, finally, for just a (usually) black screen, try:

GRUB_CMDLINE_LINUX_DEFAULT=quiet   #Don't show Ubuntu bootup text
GRUB_CMDLINE_LINUX="console=tty12" #Don't show kernel text
After editing the file, you need to run update-grub.

sudo update-grub

#xcursortheme

sudo update-alternatives --config x-cursor-theme

#Selecting I/O Schedulers
Prior to Ubuntu 19.04 with Linux 5.0 or Ubuntu 18.04.3 with Linux 4.15, the multiqueue I/O scheduling was not enabled by default and just the deadline, cfq and noop I/O schedulers were available by default.
For Ubuntu 19.10 with Linux 5.0 or Ubuntu 18.04.3 with Linux 5.0 onwards, multiqueue is enabled by default providing the bfq, kyber, mq-deadline and none I/O schedulers. For Ubuntu 19.10 with Linux 5.3 the deadline, cfq and noop I/O schedulers are deprecated.
With the Linux 5.0 kernels, one can disable these and fall back to the non-multiqueue I/O schedulers using a kernel parameter, for example for SCSI devices one can use:
scsi_mod.use_blk_mq=0
..add this to the GRUB_CMDLINE_LINUX_DEFAULT string in /etc/default/grub and run sudo update-grub to enable this option.
Changing an I/O scheduler is performed on a per block device basis. For example, for non-multi queue device /dev/sda one can see the current I/O schedulers available using the following:
cat /sys/block/sda/queue/scheduler
noop deadline [cfq]
to change this to deadline use:
echo "deadline" | sudo tee /sys/block/sda/queue/scheduler
For multiqueue devices the default will show:
cat /sys/block/sda/queue/scheduler 
[mq-deadline] none
To use kyber, install the module:
sudo modprobe kyber-iosched
cat /sys/block/sda/queue/scheduler 
[mq-deadline] kyber none
and enable it:
echo "kyber" | sudo tee /sys/block/sda/queue/scheduler
To use bfq, install the module:
sudo modprobe bfq
cat /sys/block/sda/queue/scheduler 
[mq-deadline] kyber none
and enable it:
echo "bfq" | sudo tee /sys/block/sda/queue/scheduler


#powerkey ignore

/etc/systemd/logind.conf to disable the power button:
HandlePowerKey=ignore
To pick up the new setting, restart logind with
sudo systemctl restart systemd-logind
#mint panel reset
1) Open up your terminal (ctrl+alt+t)
2) Run the following command in the terminal:
	gsettings reset-recursively org.cinnamon (THIS IS FOR CINNAMON)
	gsettings reset-recursively org.mate.panel (THIS IS FOR MATE)

#Ubuntu Spying


sudo apt remove popularity-contest

ubuntu-report -f send no
sudo apt purge ubuntu-report popularity-contest apport whoopsie

#ubuntu de-snap to deb

snap remove gnome-calculator gnome-system-monitor gnome-characters gnome-logs
sudo apt install gnome-calculator gnome-system-monitor gnome-characters gnome-logs

#dell factory sources bionic

deb http://dell.archive.canonical.com/updates/ bionic-dell public
# deb-src http://dell.archive.canonical.com/updates/ bionic-dell public

deb http://dell.archive.canonical.com/updates/ bionic-dell-loki-n3-v3-whl public
# deb-src http://dell.archive.canonical.com/updates/ bionic-dell-loki-n3-v3-whl public

deb http://dell.archive.canonical.com/updates/ bionic-dell-service public
# deb-src http://dell.archive.canonical.com/updates/ bionic-dell-service public

deb http://oem.archive.canonical.com/updates/ bionic-oem public
# deb-src http://oem.archive.canonical.com/updates/ bionic-oem public

### THIS FILE IS AUTOMATICALLY CONFIGURED ###
# You may comment out this entry, but any other modifications may be lost.
deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main


#network manager restart

 sudo service network-manager restart

#cpu speed 
watch -n.1 "cat /proc/cpuinfo | grep \"^[c]pu MHz\""


#amdgpu


I have been plowing the internet for days over this issue. Had similar setup and problems. I chanced upon here about using DRI_PRIME=1 before any commands to use your dedicated GPU. So I tried

export $DRI_PRIME=1
and this worked. So I added a line it in /etc/environment,  DRI_PRIME=1 , rebooted and now System Details shows my AMD card as default.

Caveat: I think this disables the Integrated Graphics and changes your system to just make use your dedicated GPU only

#virtualbox
sudo usermod -a -G vboxusers nameless


#panel-restart
touch panel-restart
echo '#!/bin/bash' > panel-restart
echo 'killall plasmashell;plasmashell &' >> panel-restart
sudo chmod +x panel-restart
sudo mv panel-restart /usr/bin/
panel-restart

#microsoft fonts
sudo apt install ttf-mscorefonts-installer

#x compositor manager

xcompmgr -r10 -f -c -F -o0.5 -D2.5

#recordfail grub

sudo sed -i "/recordfail_broken=/{s/1/0/}" /etc/grub.d/00_header
sudo update-grub

#bashrc

nano ~/.bashrc

export PS1="\[\033[38;5;6m\]\u\[$(tput sgr0)\]\[\033[38;5;7m\]@\h\[$(tput sgr0)\]:\[$(tput sgr0)\]\[\033[38;5;11m\][\w]\[$(tput sgr0)\]: \[$(tput sgr0)\]"

#resume
/etc/initramfs-tools/conf.d/resume
RESUME=none

#baloo index toggle

balooctl disable


#xfwm4 window manager shortcuts

xfwm4-settings
xfwm4-tweaks-settings
xfwm4-workspace-settings

#ksuperkey 

sudo apt-get install git gcc make libx11-dev libxtst-dev pkg-config
git clone https://github.com/hanschen/ksuperkey.git
cd ksuperkey
make
sudo make install

ksuperkey -e 'Super_L=Super_L|F1'

#amdgputweaks

sudo nano /usr/share/X11/xorg.conf.d/20-amdgpu.conf


Section "Device"
Identifier "AMD"
Driver "amdgpu"
Option "TearFree" "true"
Option "DRI" "3"
Option "SWCursor" "True"
Option "AccelMethod" "glamor"
EndSection


#firasans

wget https://github.com/bBoxType/FiraSans/archive/master.zip
unzip master.zip
sudo mkdir -p /usr/share/fonts/opentype/fira
sudo mkdir -p /usr/share/fonts/truetype/fira
sudo find FiraSans-master/ -name "*.otf" -exec cp {} /usr/share/fonts/opentype/fira/ \;
sudo find FiraSans-master/ -name "*.ttf" -exec cp {} /usr/share/fonts/truetype/fira/ \;


#ramdisk tmpfs
sudo nano /etc/fstab

none /tmp/ram/ tmpfs nodev,nosuid,noatime,mode=1777,size=4096M    0    0

#firefox tmpfs

Az about:config alatt létre kell hozni a következő karakterláncot: 
 
browser.cache.disk.parent_directory 
 
Értéknek pedig a következőt kell megadni:

/tmp/ram/firefox

#chromium tmpfs

--disk-cache-dir=/tmp/ram/chromium
 
Például: chromium-browser %U --disk-cache-dir=/tmp/ram/chromium
Például: /usr/bin/vivaldi-stable %U --disk-cache-dir=/tmp/ram/vivaldi

#Plasma Animations 
nano ~/.config/plasmarc

[Units]
longDuration=200

#Kubuntu Backports

sudo add-apt-repository ppa:kubuntu-ppa/backports
sudo apt-get update

#nocsd for GTK3 under KDE

gtk3-nocsd

#hide snap folder from home

echo snap >> ~/.hidden

#swappiness 

sudo nano /etc/sysctl.conf

vm.swappiness = 10

#swappiness on the fly

sudo sysctl vm.swappiness=10

#APT_FAST

sudo add-apt-repository ppa:apt-fast/stable
sudo apt-get update
sudo apt-get install apt-fast

#GPU DRIVERS
#Nvidia:

sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update

#Figyelem! Telepítés előtt ellenőrizd a kompatibilitást.

sudo apt install nvidia-driver-418 libnvidia-gl-418 libnvidia-gl-418:i386

Jelenleg a 4.18 elérhető. Nincs itthon nVidia, szóval nem tudtam tesztelni.

#AMD/Intel

#Vega nélkül:

sudo add-apt-repository ppa:paulo-miguel-dias/pkppa
sudo apt update && sudo apt upgrade


#Vega támogatással:

sudo add-apt-repository ppa:paulo-miguel-dias/mesa
sudo apt update && sudo apt upgrade


#Alternatív Ubuntu-X Team Stable forrás:

sudo add-apt-repository ppa:ubuntu-x-swat/updates
sudo apt update && sudo apt upgrade

#Játékokhoz 32bit mesa

sudo apt install libgl1-mesa-glx:i386 libgl1-mesa-dri:i386

#Figyelem 18.04 vagy újabb Ubuntu kiadásra van szükség!

#Vulkan telepítése

#nVidia

sudo apt install libvulkan1 libvulkan1:i386

#AMD/Intel

sudo apt install mesa-vulkan-drivers mesa-vulkan-drivers:i386

#I/O Scheduler WORKING

sudo nano /etc/udev/rules.d/60-schedulers.rules

# set deadline scheduler for non-rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="deadline"

# set cfq scheduler for rotating disks
ACTION=="add|change", KERNEL=="sd[b-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="deadline"

echo deadline | sudo tee /sys/block/sda/queue/scheduler

#KDE NEON REPO
1. KDE neon key hozzáadása a rendszerhez

wget -qO - 'http://archive.neon.kde.org/public.key' | sudo apt-key add -

2. KDE neon források felvétele a listába

sudo nano /etc/apt/sources.list.d/neon.list


#KDE neon repo for bionic
deb http://archive.neon.kde.org/user bionic main
deb-src http://archive.neon.kde.org/user bionic main

3. sudo apt update

4. sudo apt install neon-desktop

Ez a lépés a Kubuntu miatt valószínűleg megszakad, de semmi pánik…

5. sudo apt --fix-broken install

A megszakadt telepítés javítása után folytassuk...

6. sudo apt-get dist-upgrade -o dpkg::options::="--force-overwrite"


7. sudo apt install neon-settings

Fontos! (disztró információk, beállítások stb.)

8. sudo apt remove plymouth-theme-kubuntu-logo plymouth-theme-kubuntu-text
sudo apt install plymouth-theme-breeze grub-theme-breeze
sudo apt remove kubuntu-wallpapers-bionic
sudo apt install plasma-workspace-wallpapers

9. sudo apt autoremove
sudo apt clean

10. sudo reboot

#ELAN Button remap

xinput set-button-map "DELL08C9:00 04F3:30C4 Touchpad" 1 0 3 4 5 6 7



#HOSTS FILE MVPS
cp /etc/hosts /home/nameless/.etchosts


Utána pedig készítsd el a szkriptet

sudo nano /etc/update-hosts.sh

Tipp: A ctrl+o mentés a ctrl+x kilépés.

#!/bin/bash
cd /tmp
wget http://winhelp2002.mvps.org/hosts.txt
rm /etc/hosts
mv hosts.txt /etc/hosts
cat /home/nameless/.etchosts >> /etc/hosts


Tedd futtathatóvá:

sudo chmod +x /etc/update-hosts.sh

Futtasd:

sudo bash /etc/update-hosts.sh


Ütemezés:

sudo crontab -e

59 23 * * * /etc/update-hosts.sh

#Hibernate Sleep Disable

sudo nano /etc/polkit-1/localauthority/90-mandatory.d/disable-suspend.pkla



[Disable suspend by default]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate
ResultActive=no

[Disable suspend for all sessions]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate-multiple-sessions
ResultActive=no

#Hibernate Sleep Disable 2

Alvás és hibernálás tiltása:

sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target


Alvás és hibernálás engedélyezése

sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target


#Ubuntu Livepatch

sudo apt update && sudo apt install snapd
sudo reboot
sudo snap install canonical-livepatch
sudo canonical-livepatch enable d3b07384d113edec49eaa6238ad5ff00 (Ez csak egy példa token)
sudo canonical-livepatch status


client-version: 9.3.0
architecture: x86_64
cpu-model: AMD A9-9420 RADEON R5, 5 COMPUTE CORES 2C+3G
last-check: 2019-06-19T11:43:46+02:00
boot-time: 2019-06-18T20:45:30+02:00
uptime: 15h29m53s
status:
- kernel: 4.15.0-52.56-generic
running: true
livepatch:
checkState: checked
patchState: nothing-to-apply
version: ""
fixes: ""

#Flatpak

flatpak install flathub org.audacityteam.Audacity && flatpak install flathub com.bitwarden.desktop && flatpak install flathub org.gimp.GIMP && flatpak install flathub org.kde.kdenlive && flatpak install flathub net.minetest.Minetest && flatpak install flathub ws.openarena.OpenArena && flatpak install flathub net.openra.OpenRA && flatpak install flathub org.qbittorrent.qBittorrent && flatpak install flathub com.spotify.Client && flatpak install flathub org.telegram.desktop && flatpak install flathub org.videolan.VLC && flatpak install flathub org.gnome.FeedReader && flatpak install flathub org.gtk.Gtk3theme.Breeze && flatpak install flathub org.gtk.Gtk3theme.Breeze-Dark

#BFQ I/O scheduler on kernel >=4.12 UBUNTU

GRUB_CMDLINE_LINUX="scsi_mod.use_blk_mq=1 quiet"

sudo nano /etc/udev/rules.d/60-scheduler.rules

# set scheduler for non-rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]", TEST!="queue/rotational", ATTR{queue/scheduler}="noop"
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="noop"

# set scheduler for rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="deadline"

cat /sys/block/sda/queue/scheduler

echo cfq | sudo tee /sys/block/sda/queue/scheduler

