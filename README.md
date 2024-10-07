following this guide for the most part - [archwiki](https://wiki.archlinux.org/title/Installation_guide)

(I USE ARCH BTW)

## Some prerequisites
1. Disable Secure Boot. It can be enabled after installation.
2. Disable Fast Boot
3. Disable Hibernation.

## Pre-installation
#### 1.1 - Preparing an installation medium
1. Install the Latest Arch-Linux ISO file from [Arch ISO](https://archlinux.org/download/)
2. Now we'll use an Etcher tool to burn the iso file into a usb.
3. You can use [Balena Etcher](https://etcher.balena.io/), [Rufus](https://rufus.ie/en/) if you are on windows.

#### 1.2 - Boot the live environment
1. Plug the USB into a USB port and go into the BIOS by smashing that F2 or F12 or Del button and change the boot priority so that the Arch USB is on number then Save and Exit
2. You'll see a grub menu with multiple things. Click Enter on the Arch linux installation medium (most probably the top one)
3. You'll see many thing just going saying ok. After Some time you'll boot into the live environment.

## Setting up an internet connection
to check  if you are connected to the internet 
~~~
ping www.google.com
~~~
if you can see packets being sent across your fine \
if not do this for wifi connection:
~~~
iwctl --passphrase "your_password" station wlan0 connect your_wifiname
~~~
 you might have to replace `wlan0` with another name because your network interface might not be named that, but it will most probably start with `wl` only \
 also replace `your_password` with your actual password and `you_wifiname` with your wifi name

## Getting the disk ready for partitioning
*NOTE*: we will be booting in UEFI mode so we will make a EFI system partition, so make sure to have your motherboard set to UEFI mode instead of Legacy. Also trying to partition from notes might be confusing but i will try to make it as simple as possible

~~~
lsblk
~~~
to see the name of your disk on which you will be installing arch linux(make sure to backup if you already have not)

*NOTE*: we will be making separate root and home partitions and will be using gdisk+cgdisk to perform our operation, **VERY VERY IMPORTANT** that you do not forget the name of the disk which you have chosen as well as the partitions you make and their number. For this installation i will be using `sdb` as my disk name (so **replace** `sdb` with your disk)

to initially clean the disk for partitioning
~~~
gdisk /dev/sdb
~~~
then press `x` for expert mode \
then press `z` to zap/clean out the entire drive \
say "yes" to any prompts for confirmation

you can do `lsblk` again to see that all the previous partitions might have gone away

## Partitioning the disk
~~~
cgdisk /dev/sdb
~~~
and this will take you to the interface where we will be performing the partition

we need to make 4 different partitions of different types and they should be as follows (follow the [youtube]() video to see how to get these 4 partitions)

| NAME | SIZE in GB | HEX CODES |
| ---- | ---------- | --------- |
| boot | 1GiB       | EF00      |
| swap | xGiB       | 8200      |
| root | xGiB       | 8300      |
| home | xGiB       | 8300      |

replace xGiB with your desired size of those 3 partitions, also minimum size for root partition should be 30GiB

then go on to navigate to WRITE ("yes" if prompted for confirmation) in the bottom bar to write all your changes and then QUIT to exit the cgdisk interface

run `lsblk` again to see the partitions you have just made

## Format the partitions
*NOTE*: make sure to write the correct partition number as me but according to what your disk and its partitions are called

for the first boot partition we will be using fat32 file system because it is the most commonly recognized file system and we will need it to boot into our system
~~~
mkfs.fat -F32 /dev/sdb1
~~~

now we will be making our swap partition and enabling it
~~~
mkswap /dev/sdb2
swapon /dev/sdb2
~~~

for the last 2 partition we will be using the linux partitioning scheme ext4
~~~
mkfs.ext4 /dev/sdb3
mkfs.ext4 /dev/sdb4
~~~

## Mounting the partitions
after we have formatted all 4 partition we will need to mount it to their respective mount points, we will also be making directories for 2 of our partitions to be mounted to

~~~
mount /dev/sdb3 /mnt
mkdir /mnt/boot
mkdir /mnt/home
mount /dev/sdb4 /mnt/home
~~~

## Ranking the mirrors
archlinux uses servers to get the packages we need to install. Having good mirrors means packages install faster as well may prevent packages from out of date due to server being out of sync. It is generally not needed because the archiso runs reflector when you connect to internet so you have a good copy of mirrorlist but it is generally better to resync the mirrors if you haven't done it in long time to remove any out of sync mirrors

lets start by making a backup of the mirrorlist to be more careful
~~~
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
~~~

i will be using a tool called rankmirrors to rank my mirrors
~~~
pacman -Sy
pacman -S pacman-contrib
rankmirrors -n 10 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
~~~

## Installing the linux kernel 
now we need to install the actual linux kernel and some base packages to the root partition
~~~
pacstrap -K /mnt base linux linux-firmware
~~~

## Generating the fstab file
we will have to create a special file called "fstab" so that on booting all the partitions are properly mounted. this certain file has all the information of each partition, its filesystem and where it needs to be mounted
~~~
genfstab -U -p /mnt >> /mnt/etc/fstab
~~~

## Now we will login into our half-installed arch linux
~~~
arch-chroot /mnt
~~~
if the already present prompt changes to something like `[root@archiso /]#` then it worked

lets also install a few packages for our ease moving forward
~~~
pacman -S nano bash-completion sudo
~~~

## Localization
we need to generate a specific file called 'locale.gen' so that the system can understand language, time format, currency symbols, etc.
~~~
nano /etc/locale.gen
~~~

*NOTE*: here you would have to find out which locale is yours but if you use the QWERTY keyboard layout and use English for your pc just scroll down till you can find
`#en_US.UTF-8 UTF_8` and then remove the hashtag`#` symbol from the front of it to **uncomment it**

*NOTE:* to exit nano text editor do **Ctrl+S** to first save and then **Ctrl+X** to exit 

then do these following steps
~~~
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
~~~

## Time
time to configure our timezone and synchronize our system clock accordingly

start by typing(dont run this command) `ls /usr/share/zoneinfo` 
here you will have to find out which timezone you belong to you can list all the countries in each folder from "zoneinfo" by running `ls` 

after you figure out your timezone run this (replace with your timezone)
~~~
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
~~~

and then run this to synchronize your system clock
~~~
hwclock --systohc 
~~~

## Setting users
we should first set a root password for more security
~~~
passwd
~~~

to add user 
~~~
useradd -m -g users -G wheel abbas
~~~

then to set user password 
~~~
passwd abbas
~~~

now we need to provide wheel support to the user we just created so that the user can have superuser(sudo/root) privileges 
~~~
EDITOR=nano visudo
~~~
scroll down to where you can see this specific line and uncomment it
`# %wheel ALL=(ALL:ALL) ALL`

## To get 32-bit support
some applications rarely, but might require 32-bit support, so its better to enable it 
~~~
nano /etc/pacman.conf
~~~
again scroll down to where you can find these two lines and uncomment them both
~~~
#[multilib]
#include = /etc/pacman.d/mirrorlist
~~~

now we need to sync our local database so that arch linux can download the multilib package which is required for 32-bit support
~~~
sudo pacman -Sy
~~~

## Installing some essential packages
now we need to install a few packages which we need for arch linux to function as we intend
~~~
sudo pacman -S base-devel dosfstools grub efibootmgr mtools networkmanager openssh linux-headers pacman-contrib konsole firefox dolphin git ark unzip
~~~
to know what all these packages do, see my arch linux installation [video]()

## Installing graphic drivers
this is a very important step because majority of what will be displayed will be with the help of a graphic card, although it doesnt matter if it is a dedicated one or an integrated GPU.
These drivers will vary upon whose card you are using

if you are using **intel** graphics(integrated also)
~~~
sudo pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel 
intel-media-driver libva-intel-driver lib32-libva-intel-driver
~~~

if you are using **AMD** graphics card(integrated or dedicated)
~~~
sudo pacman -S mesa lib32-mesa xf86-video-amdgpu vulkan-radeon 
lib32-vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver
~~~

if you are using **Nvidia** graphics card
~~~
sudo pacman -S nvidia libglvnd nvidia-utils opencl-nvidia lib32-libglvnd 
lib32-nvidia-utils lib32-opencl-nvidia nvidia-settings
~~~
\
<br />
*NOTE*: You can run into problems when it comes to graphic drivers especially Nvidia's proprietary drivers, so if you come across any such problem leave a comment on this [video]() and i shall reach back to you with a solution as i have a few different methods to fix driver issues

## Configuring the grub bootloader
we did install grub package but it doesnt work or get installed directly out of the box so we need to make a few changes before that, and this is a **very important step** because without a bootloader like GRUB which we will be using, our computer wouldn't know what operating system to load

we will now make use of the first partition we made here
~~~
mkdir /boot/EFI
mount /dev/sdb1 /boot/EFI
~~~

now to properly install grub
~~~
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
~~~
then 
~~~
cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
~~~

now to compile grub so that it applies all the changes
~~~
grub-mkconfig -o /boot/grub/grub.cfg
~~~

## Installing the Desktop Environment
a desktop environment is exactly what the name means, it is the environment of your desktop, in simpler terms it is the graphical user interface that provides a way and interface for you to interact with your computer. for this installation i will be using "KDE Plasma" desktop environment which is the simplest according to me for a new user and that is what i daily drive as well as it is the one i am most accustomed to

on KDE Plasma we will be using the "Xorg" display server based on the "X11" protocol, too much to digest, but to put it in layman terms X11 or Xorg is the backbone which we will be using to do stuff like displaying windows, handling inputs like keyboard and mouse, drawing graphics, etc.

also we will be installing the  "sddm" display manager which basically is just the login screen to enter our desktop environment or switch between them

to install the following run
~~~
sudo pacman -S xorg plasma sddm
~~~

## Enabling a few startup services
it would be helpful for us if we can have some applications or services which start as we start our system so that we dont have to manually do it every time

we will first enable "Network Manager" because of obvious reasons
~~~
systemctl enable NetworkManager
~~~

then  we will also enable "fstim.timer" which is **only applicable** if you have a SSD because it helps the SSD inform which blocks of data are useless and can be wiped off, thus increasing the SSD's efficiency and longevity   
~~~
systemctl enable fstrim.timer
~~~

next we have to enable "sddm" so that we can be able to use the login screen to login 
~~~
sudo systemctl enable sddm
~~~

## Exiting and restarting 
~~~
exit
~~~
so that we can come out of our arch linux, which we previously logged in to and then installed arch completely

now we have to unmount all the partitions we mounted so that when we remove our installation media we will not get any error when logging in the next time and arch linux can mount all the partitions to the system disk directly 
~~~
umount -R /mnt
~~~

now 
~~~
reboot
~~~
do this and  then when the pc shuts off completely, remove the installation media and boot into your new arch linux and conquer the world

*NOTE*: if u are installing this on a virtual machine do `shutdown` and then go to your VM settings and remove the iso image so that on the next startup it can boot into arch linux and not the live environment we were just in

## There you have it! A hassle free and easy to understand arch linux installation guide without all the jargon you would have got otherwise

 
