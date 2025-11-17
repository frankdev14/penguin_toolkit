# Fedora Post-Install

After performing a clean installation of Fedora, with this guide you will be able to have your Linux system up and running quickly, updated, and ready for Nvidia. It should be noted that I am an Nvidia user, so if you do not have a desktop with Nvidia graphics, you may need to do something different than what I did.

1. First of all, we need speed up our dnf package manager:
~~~
echo 'fastestmirror=1' | sudo tee -a /etc/dnf/dnf.conf
echo 'max_parallel_downloads=10' | sudo tee -a /etc/dnf/dnf.conf
echo 'deltarpm=true' | sudo tee -a /etc/dnf/dnf.conf
cat /etc/dnf/dnf.conf
~~~

2. After that, we can proceed to install third party repos, I fall in love with RPMFusion and Terra.

~~~
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

sudo dnf install --nogpgcheck --repofrompath 'terra,https://repos.fyralabs.com/terra$releasever' terra-release
~~~

3. Now, need upgrade app-stream metadata:
~~~
sudo dnf group upgrade core
sudo dnf4 group install core
sudo dnf group install development-tools
sudo dnf install git-lfs
~~~


4. Update the system and reboot:

~~~
sudo dnf -y update
~~~

5. After reboot, we can proceed to check firmware updates:

~~~
sudo fwupdmgr refresh --force
sudo fwupdmgr get-devices
sudo fwupdmgr get-updates
sudo fwupdmgr update
~~~

6. Add Flathub to our flatpak. First command will enable a system flathub install and second will enable on user home:

~~~
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

flatpak remote-add --user --if-not-exists flathub https://flathub.org/repo/flath>
~~~

7. Optional - Enable Appimageand install a manager:
~~~
sudo dnf in fuse

flatpak install it.mijorus.gearlever
~~~

8. Installing our GPU drivers. This part depends on your system. If you have an NVIDIA graphics card, as I do, you can continue with these commands.

~~~
sudo dnf install akmod-nvidia

sudo dnf install xorg-x11-drv-nvidia-cuda
~~~

- Now wait 5 min and check if the kernel is built: modinfo -F version nvidia

- Reboot after this 5 mim.

9. Optional - Installing a GPU Fan controller.

~~~
sudo dnf copr enable ilyaz/LACT

sudo dnf install lact

sudo systemctl enable --now lactd
~~~

- Now you should search Lact in your applauncher and config the fans on thermal section. With this step You can even configure the graphics and overclock or whatever you prefer. It's like MSI Afterburner.

10. Enable a proper multimedia playback.
~~~
sudo dnf4 group install multimedia
sudo dnf swap 'ffmpeg-free' 'ffmpeg' --allowerasing
sudo dnf upgrade @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin
sudo dnf group install -y sound-and-video
~~~

11. Enable video acceleration.
~~~
sudo dnf install ffmpeg-libs libva libva-utils
sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264
sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1

Enable the plugin on firefox
~~~

12. Remove fedora default page.
~~~
sudo rm -f /usr/lib64/firefox/browser/defaults/preferences/firefox-redhat-default-prefs.js
~~~

13. Setup cloudflare DNS.
~~~
sudo mkdir -p '/etc/systemd/resolved.conf.d' && sudo -e '/etc/systemd/resolved.conf.d/99-dns-over-tls.conf'

[Resolve]
DNS=1.1.1.2#security.cloudflare-dns.com 1.0.0.2#security.cloudflare-dns.com 2606:4700:4700::1112#security.cloudflare-dns.com 2606:4700:4700::1002#security.cloudflare-dns.com
DNSOverTLS=yes
Domains=~.
~~~

14. For dualboot system fix hour exec this.
~~~
sudo timedatectl set-local-rtc '0'
~~~

15. Wayland is powerfull but need this tweaks.
~~~
sudo grubby --update-kernel=ALL --args="nvidia-drm.modeset=1"
~~~

16. Boot time optimizer.
~~~
sudo systemctl disable NetworkManager-wait-online.service
~~~

17. Install zip extensions.
~~~
sudo dnf install -y unzip p7zip p7zip-plugins unrar
~~~

18. Clean the system.
~~~
sudo dnf remove *libreoffice*
sudo dnf autoremove
sudo dnf clean all
~~~

19. Install partition manager and edit fstab.

~~~
sudo dnf install partitionmanager

sudo nano /etc/fstab

Save UUID for the next step:
sudo blkid

Add this line for every partition as you wish to automount and enable it for development, game partition, etc.

UUID=your_UUID /mnt/Games ext4 defaults,user,exec,rw 0 2

Mount the new fstab: sudo mount -a

When all is mounted, you should give the right permissions to every folder that you mount:
sudo chown user:user /mnt/Games
~~~
___

20. Extras that I do but not included into this post-install:
~~~
1 - Install apps like steam, discord, protonup-qp, protontricks, vlc, brave, zen-browser etc.
2 - Setup gaming system for play, by default my fedora KDE uses falcond but I strongly recommend that you install GE Proton with ProtonUp-QT.
3 - Check the system and test games.
4 - Install zsh, oh my zsh & set default ZSH. Install plugins and put the dotfiles into your directory.
5 - Install starship before you reboot to get zsh works.
6 - Configure git, ssh, and gpg to work with GitHub, and use it with a GitHub noreply email address for maximum privacy.
7 - Install uv for python management and library management.
8 - Install podman as docker replacement.
9 - Setup all web pages and login into all your accounts.
10 - Put my ".zshrc" config file into your home directory and open it. Install all zsh plugins that you dont have in my zsh config and restart the shell.
11 - Dont forget configure your system like displays, language, etc.
~~~

## With that, my Fedora is ready for everyday use! Now I can customize KDE and make it my own!