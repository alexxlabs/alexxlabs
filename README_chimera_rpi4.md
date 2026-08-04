pyinfra ???: https://docs.pyinfra.com/en/3.x/getting-started.html

# TODO lua based !!!: https://github.com/myFavShrimp/arc

# ==================== preparations before install ===============================
boot from chimera rpi flash, login as 'root' : 'chimera'

start DHCPCD:

# dinitctl start dhcpcd

'end0' brings UP and get its IP address from dhcp server:

after that edit:

# nano /etc/ssh/sshd_config

allow root login and password auth and:

# dinitctl start sshd

after that you can connect to rpi4 from your PC by ssh
and perform comfort install by ssh (with copy/paste commands)

# ================================================================================

Let’s assume that the target disk drive is /dev/sda.
Let’s start with wiping everything on it:

# wipefs -a /dev/sda

# ================================================================================

Once you have wiped the drive, the easiest way to initialize a partition table
and create partitions is with the cfdisk TUI program:

# cfdisk /dev/sda

Required partitions:
Partition table: MBR/GPT

For Raspberry Pi 4, you can use GPT, while MBR will also work.
!!! You will need to have a partition for '/boot' **as the first partition**

 1) /boot partition
		On MBR this needs to be marked **bootable**
			lsblk -o label ( see labels )
			lsblk -o UUID,NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL,MODEL
			Set label:
				for fat/vfat (using dosfstools):		fatlabel /dev/XXX <label>
				for fat/vfat (using mtools):			mlabel -i /dev/XXX ::<label>
				for ext2/ext3/ext4 filesystems:			e2label /dev/XXX <label>
				for swap (first you need to swapoff):	swaplabel -L <label> /dev/XXX
		On GPT, it needs to be of type
			Microsoft basic data
			or EFI System,
		or it will not be found.

 2) Swap
		This is not required, but you might want one, depending on your
		system RAM and other requirements (e.g. hibernation).
		The partition type should be Linux swap. A good amount is at least 4 gigabytes.
		The old guidelines for swap size based on your physical RAM no longer apply these days.

 3) /root ( ext4 - Root filesystem )

 4) /home ( the rest of HDD )

final partitioning will be:

/dev/sda1 - /boot	[ EFI ]				mkfs.vfat /dev/sda1
/dev/sda2 - swap	[ linux swap ]		mkswap /dev/sda2
/dev/sda3 - /	[ ext4 ]				mkfs.ext4 /dev/sda3
/dev/sda4 - /home [ ext4 ]				mkfs.ext4 /dev/sda4

Installing:

# mkdir -p /media/root
# mount /dev/sda3 /media/root
# mkdir -p /media/root/boot
# mount /dev/sda1 /media/root/boot
# mkdir -p /media/root/home
# mount /dev/sda4 /media/root/home
# chmod 755 /media/root
# chimera-bootstrap /media/root
# chimera-chroot /media/root
# date YYYYMMDDHHmm	[ check - and - if incorrect - then adjust ]
# apk update
# apk upgrade --available
# apk add base-rpi
# apk add linux-rpi
# swapon /dev/sda2
# genfstab -U / >> /etc/fstab

You also have to edit /boot/cmdline.txt and change the root option to the UUID
of the partition on the HDD USB disk.
I got that by running on the machine with the USB drive plugged in.
Replace the PARTUUID to the result of command:

# lsblk -o name,partuuid

root=PARTUUID=< lsblk -o name,partuuid > rootfstype=ext4 ro rootwait console=ttyAMA0,115200 console=tty1 dwc_otg.lpm_enable=0 quiet splash

(note, I prefer using PARTUUID to point to the drive, rather than /dev/sda3,
 as it will not change. If you used sda3, it may change if you plug different
 USB storage devices into your pi)

# passwd root
# update-initramfs -c -k all

### !!!!!!!!!!!!!!!!!!!!!!!!!!! POST INSTALLATION tasks: AFTER REBOOT !!!!!!!!!!!!!!!!!!!!!!!!!!!

Enable and persist services:

# dinitctl enable dhcpcd
# dinitctl enable chrony

Repositories
By default, you will get packages from the main repository.
However, Chimera also has packages in the user repository,
which contains extra software unsuitable for main.

The user repository has software that does not meet the general criteria regarding stable
versioning, dependency vendoring, and so on. That means software in user may not be of the
same quality level as software in main. Software in user may additionally not be available
on all tier 1 architectures.

You do not need to manually change any config files to enable these.
The simplest way to get it is the following:

# apk add chimera-repo-user

Afterwards simply refresh your indexes:

# apk update

Immediately after installation, you will typically only have the root user.
You should not be using root as your regular user. Creating one is easy:

# useradd alexxlabs

Set a password, so you can log in:

# passwd alexxlabs

While at it, you might want to add your user to some groups.
This is not strictly necessary. Some groups that might be useful:

 - wheel is the local administrator group
 - kvm will let your user handle virtual machines
 - plugdev will let you access removable devices where there is no other
 	(e.g. policy-based) mechanism (usually unnecessary)
 - audio and video should not be necessary (elogind and similar provide better,
 	more secure mechanisms that do not require giving your user raw device access)
	except in very specific cases

To add your user to a group or groups:

# usermod -a -G wheel alexxlabs

You should avoid adding your user to groups you do not strictly need.

### change default shell for alexxlabs

# apk add bash
# chsh -s /bin/bash alexxlabs


### ================= SSH service setup =========================

 - !!! Private Keys (id_rsa, id_ed25519): Must be strictly 600 (-rw-------).
 
Rule of Thumb: If files are not accessible or permissions are too open,
the SSH client will aggressively block connections to protect your infrastructure.

# chmod 700 /root/.ssh
# chmod 644 /root/.ssh/authorized_keys

 - The main directory /home itself must always be 755 and owned by root:root
# chown root:root /home
# chmod 755 /home

 - Only the owner can read, write, and execute; best for high security and private data.
# chown alexxlabs:alexxlabs /home/alexxlabs
# chmod 700 /home/alexxlabs

 - .ssh Directory: Must be set to 700 (drwx------).
# mkdir -p /home/alexxlabs/.ssh
# chmod 700 /home/alexxlabs/.ssh
# chown -R alexxlabs:wheel /home/alexxlabs/.ssh

 - Public Keys & authorized_keys: Should be set to 644 (-rw-r--r--).
# touch /home/alexxlabs/.ssh/authorized_keys
# chmod 644 /home/alexxlabs/.ssh/authorized_keys
# chown alexxlabs:wheel /home/alexxlabs/.ssh/authorized_keys

# cat /root/.ssh/authorized_keys > /home/alexxlabs/.ssh/authorized_keys

maybe fine tune '/etc/ssh/sshd_config' and:

# dinitctl enable sshd
# dinitctl restart sshd

# ===============================================================

# echo pi > /etc/hostname
# ln -sf /usr/share/zoneinfo/Europe/Chisinau /etc/localtime
# dinitctl enable syslog-ng

### rc.local

The system administrator can create the script /etc/rc.local,
which is run after early init is done, and either before or in
parallel with regular service startup.
This can be used to run things that are unfit for regular service handling.

# ================================================================================

# https://www.raspberrypi.com/documentation/computers/config_txt.html

Instead of the BIOS found on a conventional PC,
Raspberry Pi devices use a configuration file called config.txt.
The GPU reads config.txt before the Arm CPU and Linux initialise.
Raspberry Pi OS looks for this file in the boot partition,
located at /boot/config.txt

# ================================================================================
# ======================= Setting up /etc/doas.conf ==============================
# ================================================================================
You need to setup your /etc/doas.conf like this:

	permit setenv {PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin} :wheel
	permit setenv { XAUTHORITY LANG LC_ALL } :wheel
	permit persist setenv {PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin} :wheel
	permit nopass :wheel as root cmd /usr/sbin/reboot
	permit nopass :wheel as root cmd /usr/sbin/poweroff

Now let's go through this line by line!

Line 1: This sets up the $PATH variable correctly, working around an issue arising from doas's
		OpenBSD origin where all additional system packages are stored in /usr/local/bin.

Line 2: This sets some variables to work around an issue arising from certain packages like
		devtools having a hard-dependency on sudo. I don't know if this is an issue on
		Gentoo but i set it anyway, to be safe.

Line 3: Setting up the persist feature for the wheel group, with the $PATH variable,
		for the same reasons as line 1. You can remove this if you don't have
		the persist feature enabled.

Line 4 & 5: This gives the wheel group permission to execute /usr/sbin/reboot
		and /usr/sbin/poweroff without typing in a password

Changing ownership and permissions of /etc/doas.conf

# chmod -c 0400 /etc/doas.conf
# chown -c root:root /etc/doas.conf

This makes /etc/doas.conf only readable by the root user, owned by the root user as well
as the root group. It might sound too stripped down, but trust me, it works!

In practice

If you're coming from just works distros like Linux Mint you might find yourself typing
reboot and poweroff. This doesn't work unless you're logged in as root, instead you need
to type doas /usr/sbin/reboot or doas /usr/sbin/poweroff

If you want to avoid typing this long ass command out, add an alias in your
users .rc file for whatever shell your using. If you're using bash,
it would be .bashrc

The alias can be whatever you want, heres an example:

alias reboot='doas /usr/sbin/reboot'
alias poweroff='doas /usr/sbin/poweroff'
# ================================================================================

