# My Arch Linux VM Installation

## Goal and environment

I am manually installing an Arch Linux ARM virtual machine for my Linux assignment, using
the Arch Wiki installation guide as my main reference.
- Installation date: October 28, 2026
- Host computer: Apple Silicon Mac
- Virtualization software: UTM
- Installation media: Archboot ARM64 ISO
- Virtual installation disk: 40 GiB, identified as /dev/vda
I used the ARM64 Archboot image linked in the assignment because my Mac uses Apple
Silicon. Some commands in the standard Arch Wiki installation guide were different than what I had to use.

## 1. Booting the installation environment

I downloaded the ARM64 ISO from:
https://release.archboot.com/aarch64/latest/iso/
I attached it to my UTM VM and started the VM. I
pressed Control + C to open a Bash prompt for the manual installation.
I ran:

```bash
whoami
```

This checks the current user. The output was root, confirming that I had admin access in
the live installation environment.

```bash
uname -m
```

This checks the architecture. The output was aarch64, confirming that I booted the ARM64
environment.

## 2. Keyboard layout

I wanted to use a US keyboard layout and tried:

```bash
loadkeys us
```

The command returned a “command not found” error. I did not change the keyboard layout at
this point. I went with the default.
The Archboot environment did not provide the loadkeys command as
expected from the standard Arch Wiki instructions.

## 3. Network connection

I checked the network configuration:

```bash
ip address
```

The virtual network interface was enp0s1, with the automatically assigned IPv4 address
192.168.64.6/24.
The VM uses a virtual Ethernet connection through UTM, so I did not need to connect to Wi-Fi
inside Linux or manually enter an IP address or subnet.
I initially entered:

```bash
ping -c archlinux.org
```

This failed because -c requires a packet count. I corrected it to:

```bash
ping -c 3 archlinux.org
```

This sends three ping requests to test Internet connectivity and name resolution. All three
requests received replies.

## 4. Identifying the installation disk

I listed the available block devices:

```bash
lsblk
```

The output showed:

| Device | Size | Purpose |
|---|---|---|
| /dev/sr0 | 469.7 MiB | Attached installation ISO |
| /dev/zram0 | 6 GiB | RAM-backed device used by the live environment |
| /dev/vda | 40 GiB | Virtual disk for the installation |

I also inspected the disks with:

```bash
fdisk -l
```

This displayed disk sizes and partition information. /dev/vda had no existing partitions listed.
I identified /dev/vda as the installation target. I will leave /dev/zram0 and the installation ISO
unchanged.

## 5. Partitioning plan

My planned layout uses a GPT partition table:

| Planned partition | Size | Type | Purpose |
|---|---|---|---|
| /dev/vda1 | 1 GiB | EFI System | Boot files, mounted at /boot |
| /dev/vda2 | 4 GiB | Linux swap | Swap space |
| /dev/vda3 | Remaining space, about 35 GiB | Linux filesystem | Root filesystem, mounted at / |

The command for opening the partition editor is:

```bash
fdisk /dev/vda
```

At this point I didn’t know what to correctly type for partitioning. I looked it up and found it was
a series of letter commands and setting sectors:
g creates a new GPT partition. n creates a partition number, last sector specifies the size of the partition, t is for what type of partition,
and the list was numbered, I chose 1 because this was my EFI System and needed one GiB.
Next, I did the same process for my swap space, changing from +1G to +4G. using 19: Linux
swap.
After that, I had the rest of the drive to partition, so I hit enter to default the number, first and last
section, and type. After these drives were created, I used p to review the partitions and w to save them.
I now needed to format the partitions, so I took the commands from the wiki but changed them to
/dev/vda(partition name)

```bash
mkfs.fat -F 32 /dev/vda1
```

```bash
mkswap /dev/vda2
```

```bash
mkfs.ext4 /dev/vda3
```

After formatting, I proceeded to mounting and ran these to mount all of them

```bash
mount /dev/vda3 /mnt
```

```bash
mount --mkdir /dev/vda1 /mnt/boot
```

```bash
swapon /dev/vda2
```

## 6. Installation

I ran with the default mirror and then ran:

```bash
pacstrap -K /mnt base linux linux-firmware
```

But modified it a little to fit arm64, after doing some research

```bash
pacstrap –K /mnt base linux-aarch64 linux-firmware
```

After this, I worked on configuration, firstly with file system mounting on startup. The command
for this is:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

I then checked it with:

```bash
cat /mnt/etc/fstab
```

After that, to interact with the system as if booted, I ran:

```bash
arch-chroot -S /mnt
```

I then ran some stuff to set time zones

```bash
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

and

```bash
hwclock --systohc
```

I then went on to generate the locales which helps when setting region specific formatting.
First I modified the text file for the locale
```bash
nano /etc/locale.gen
```
then removed the comment hash from en_US.UTF-8 UTF-8
Then generated the enabled locale
```bash
locale-gen
```
And create and modify the configuring file

```bash
nano /etc/locale.conf
```

I ran into a problem here and realized from the command not found error that I needed to
download nano, so I ran:

```bash
pacman -S nano
```

Another error here, error occurred it couldn’t download the nano because of a keyring issue, so I
ran

```bash
pacman-key --init
```

and then

```bash
pacman-key --populate archlinuxarm
```

this responded with another error, so I looked at what to do.
I exited the linux terminal, and went back to the live installer. At this point I was kinda lost, but
realized after some searching I should have added a archlinuxarm-keyring to my pacstrap for the
kernel installation. So I ran

```bash
pacstrap /mnt archlinuxarm-keyring
```

and this downloaded. I went back to the arch terminal and kept going with the guide, but first ran

```bash
pacman-key --init
```

and

```bash
pacman-key --populate archlinuxarm
```

I then tried to install nano and it worked! So I ran

```bash
nano /etc/locale.conf
```

I got another error, a cannot recognize terminal type error, so I looked around and saw that if I
ran

```bash
TERM=xterm nano /etc/locale.conf
```

This opened a empty text file to which I put

```text
LANG=en_US.UTF-8
```

Like the guide says, and then saved
Keyboard layout was skipped because I didn’t set one
Went to network config

## 7. Network Config and Final Booting

For the network config I first had to set a hostname, I kept running into the issue of having to
type TERM=xterm before my nano, so I ran

```bash
export TERM=xterm
```

then

```bash
nano /etc/hostname
```

called it thodge-arch
Next for booting, I picked grub. This was one of the recommended ones for arm64.
I entered a few commands that I had to search for

```bash
pacman -S grub efibootmgr
```
This downloaded the actual grub manager itself

```bash
grub-install --target=arm64-efi --efi-directory=/boot --bootloader-id=GRUB
```
This configured the grub install so that it was targeted for arm64 and ran at boot on EFI Partition

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
This pulled the boot menu, so I could verify that the bootloader finds the kernel

Then I exited. I unmounted using:

```bash
umount -R /mnt
```

And then rebooted

```bash
reboot
```

then turned the VM off
I cleared the CD/DVD Drive and then turned it back on
Logged in as root, with password

## 8. GUI and Additions

For the gui and other things, I had to secure my internet, because after the reboot I had no
connection. To do this I ran (from root):
```bash
nano /etc/systemd/network/20-wired.network
```
To open a network config file, then
```INI
[Match]
Name=enp0s1

[Network]
DHCP=yes
```
Which told the system network to automatically get settings from virtual ethernet VM interface
I started networking and DNS, and made them automatically start at each boot
```bash
systemctl enable --now systemd-networkd systemd-resolved
```
Then I connected the system DNS to the resolver
```bash
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

I chose Xfce for my desktop. It is lightweight, and easy to install.
I ran:
```bash
sudo pacman -S xfce4 xfce4-goodies
```
and this installed the gui
I then created a account for myself and codi (Ran as root)
```bash
useradd -m -s /bin/bash taihodge
```
did the same for codi, replacing taihodge with code

```bash
useradd -m -s /bin/bash codi
```
I did my own password and codi's password with this command
```bash
passwd codi
```
Then I needed to give them sudo permissions so I added them to the wheel group. (Ran as root)
```bash
usermod -aG wheel taihodge
```
(did same with codi)
then I modified the sudo config that existed already
```bash
EDITOR=nano visudo
```
and took away the leading # that commented it out
%wheel ALL=(ALL:ALL) ALL
For another shell I used zsh:
```bash
sudo pacman -Syu zsh
```
```bash
sudo chsh -s /bin/zsh taihodge
```
then to change it's color:
```bash
sudo nano ~/.zshrc
```
I then wrote this in the text portion:
```bash
PROMPT='%F{green}%n@%m%f %F{blue}%~%f %# '
```
```bash
sudo pacman -S openssh
```
and then enabled it to launch every time I powered on
```bash
sudo systemctl enable --now sshd
```
I then checked if I had a gui boot enabled (display manager)
I had lightDM already running and installed. I rebooted and saw that after a few seconds it went to the graphical login
I then added aliases, some recommended ones were below
I used 
```bash
sudo nano ~/.zshrc
```
and added:
```bash
# Show detailed file information, including hidden files
alias ll='ls -lah --color=auto'

# Move up one directory
alias ..='cd ..'

# Clear the terminal
alias cls='clear'

# Show IP addresses in a compact format
alias myip='ip -br address'

# Update the system
alias update='sudo  -Syu'

# Show available disk space
alias space='df -h'

# Prompt before deleting files
alias rm='rm -i'
```
## 9. Other adds

I also added Firefox as a browser extension to my linux gui
To download:
```bash
pacman -S firefox
```
then to I selected between jack and pipe wire jack, which means it provided JACK compatibility through Pipewire (I typed 2). After that I had to select a provider for ttf-font. After doing some research I learned that ttf-dejavu would suit what I needed (I typed 5).





## References

- Arch Wiki installation guide: https://wiki.archlinux.org/title/Installation_guide
- Archboot: https://archboot.com/
- ARM64 installation images: https://release.archboot.com/aarch64/latest/iso/
