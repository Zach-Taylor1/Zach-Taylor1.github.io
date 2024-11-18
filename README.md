# Zach-Taylor1.github.io
**Arch Linux with UTM on Mac M2**

Download archlinux-2024.11.01-x86_64.iso

Set up VM in UTM with the ISO
	
	Allocated default amount of cores, 8GB of RAM, and 24GB of capacity

Boot up VM and select Arch Linux Install Medium
Set the font with this command:

	setfont ter-v16n

Ran this command to verify the boot mode:

	cat /sys/firmware/efi/fw_platform_size
	It returned 64 which is the best option according to the wiki as it is booted in UEFI mode and has a 64-bit x64 UEFI

Verified network connection with this command:

	ping archlinux.org
	It returned a packet every second of 64 bytes, so the network is connected

Checked the clock with this command:

	timedatectl
	Saw that it was incorrect and needed to be updated

Updated the clock to the correct timezone with this command:

	timedatectl set-timezone America/Chicago

Identified connected disks with:

	fdisk -l

Partition disk with fdisk:

	fdisk /dev/sda
	g # Sets GPT partition table
	n # Create new partition, left the number to default (1) and start as default (2048), but changed the end to (+2G)
	t # To change the type from Linux filesystem to EFI System (1)
	n # Create a second partition as Linux filesystem with all default settings
	w # To write the changes

Formatted the root partition (/dev/sda2):

	mkfs.ext4 /dev/sda2

Formatted the EFI partition (/dev/sda1):

	mkfs.fat -F 32 /dev/sda1

Mount the file systems:

	mount /dev/sda2 /mnt
	mount --mkdir /dev/sda1 /mnt/boot

Install essential packages:

	pacstrap -K /mnt base linux linux-firmware networkmanager vim nano man-db man-pages texinfo 
	Added the extras because the base system does not include them, and they are necessary for a fully functional system

Configure system with fstab:

	genfstab -U /mnt >> /mnt/etc/fstab

Chroot into the file system:

	arch-chroot /mnt

Time (but for real this time):

	ln -sf /usr/share/America/Chicago /etc/localtime
	hwclock --systohc

Localization
Here I set the language and locale:

	locale-gen 
	vim /etc/locale.gen

		Scroll to en_US.UTF-8 UTF-8
		Place cursor on the # for the line
		Press control V to select it, then x to delete it
		Exit vim and save by using the :wq command

	echo 'LANG=en_US.UTF-8' > /etc/locale.conf

To write the LANG variable to the newly created /locale.conf file
Network configuration:

	echo ‘zach-arch’ > /etc/hostname

Change root password

	passwd

Boot loader

	Had issues so needed to update the keyrings in order to trust the keys for the grub install:
		pacman-key --init
		pacman-key --populate archlinux

I went with grub because it is the best option period

	pacman -S grub efibootmgr
	grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
	grub-mkconfig -o /boot/grub/grub.cfg

Exit chroot:

	exit

Unmount partitions

	umount -R /mnt

Reboot:

	reboot
	
	This is also where you change the location disk from the iso to cd/dvd 
	(I used UTM, so go back to the main UTM application and there is a dropdown menu at the bottom of the selected VM for you to do this)

NetworkManager stuff

	systemctl enable NetworkManager
	systemctl start NetworkManager

Desktop Environment

First update the system:

	pacman -Syu

Gnome Install:

	pacman -S gnome gnome-extra
	systemctl enable gdm
	reboot

Create user accounts

	Open console

Adding users:

	useradd -m -G wheel,users -s /bin/ zach
	useradd -m -G wheel,users -s /bin/bash codi
	useradd -m -G wheel,users -s /bin/bash justin

Setting passwords:

	passwd zach
	passwd justin
		GraceHopper1906
	passwd codi
		GraceHopper1906

Setting passwords to be changed on login

	chage -d 0 justin
	chage -d 0 codi

Visudo

	pacman -S sudo vi
	visudo /etc/sudoers

		Scroll to # %wheel ALL=(ALL) ALL and remove the comment via control v, then x
		:wq to write and quit

Download alternative shell

	pacman -S zsh

Add ssh

	pacman -S openssh

Setting aliases in bash and zsh

	vim ~/.bashrc

Added two new lines:

	alias c=‘clear’
	alias update=‘sudo pacman -Syu’
		Hit esc key followed by :wq to write and quit

	vim ~/.zshrc

Added two new lines:

	alias c=‘clear’
	alias update=‘sudo pacman -Syu’
		Hit esc key followed by :wq to write and quit

Color coding the terminal

	vim ~/.bashrc

Added a new line:

	export PS1="\[\033[01;32m\]\u\[\033[00m\]\[\033[01;35m\]@\[\033[00m\]\[\033[01;36m\]\h \[\033[01;36m\]\w\[\033[00m\] \[\033[01;33m\]$ "

		Hit esc and then :wq to write and quit
		Note that these changes made the terminal very high contrast

Setting changes to terminal

	source ~/.bashrc
	source ~/.zshrc
