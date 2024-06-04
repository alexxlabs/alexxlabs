# ================ fs-joyent failed with exit status 95 ===========
# The error was related to running out of space and my dump device was not big enough,
# so here are the steps to solve it:
# Add a new disk to your Triton instance and boot it using a SmartOS usb and select
# rescue mode then execute:
# $ zpool import zones
# $ zfs destroy zones/dump
# $ zfs create -V <your_ram_amount>G zones/dump
# $ dumpadm -y -d /dev/zvol/dsk/zones/dump 
# $ reboot
#
# ========================= Manage SWAP ===========================
#
# A quick check using 'swap -sh' will show how much has been allocated vs available.
# For more detailed look: 'echo ::memstat | mdb -k'
#
The swap zvol gets created at boot time to match the size of DRAM because that's how SmartOS expects to run.
When you increase the DRAM you should increase the swap zvol accordingly.
1. Reboot into noinstall mode.
2. zpool import zones
3. zfs set volsize=<DRAM_SIZE>G zones/swap
4. zpool export zones
5. reboot
# ==================================================================
# Platform Image Management
# https://blog.brianewell.com/installing-smartos/
# https://wiki.smartos.org/remotely-upgrading-a-usb-key-based-deployment/#using-piadm-with-a-bootable-pool

2021 Update: SmartOS now supports platform images installed directly to zones/boot,
eliminating the need for a dedicated boot device.
This can be accomplished using the following command on the global zone:
[root@gz ~]# piadm bootable -e zones
This will take a while as it downloads and installs the latest available platform image.

Platform images can be updated using the following commands:
[root@gz ~]# piadm install <platform image identifier> zones
[root@gz ~]# piadm activate <platform image identifier> zones

Usage: piadm [-v] <command> [command-specific arguments]

    piadm activate|assign <PI-stamp> [ZFS-pool-name]
    piadm avail
    piadm bootable [-d] [-e [-i <source>]] [-r] [ZFS-pool-name]
    piadm install <source> [ZFS-pool-name]
    piadm list <-H> [ZFS-pool-name]
    piadm remove <PI-stamp> [ZFS-pool-name]
    piadm update [ZFS-pool-name]

# High performance scp
To copy a large file (1GB or more) over from one server to another use:
scp -c arcfour target username@server:/destination
this significant increases the transfer speed by at least 30%

# add/increase ZPOOL
to view disks: diskinfo

# expand zpool capacity:
zpool get autoexpand zones
zpool set autoexpand=on zones
zpool online -e <ZPOOL_NAME> <DEVICE_NAME_FROM_DISKINFO>
# extend zfs
zfs get volsize,reservation zones/voidlinux_tmp
zfs set volsize=20G zones/voidlinux_tmp
// ??? zfs set quota=20g zones/voidlinux_tmp

# cannot import ‘rpool’: I/O error

Если при импорте пула получаем такое сообщение

$ # zpool import -f rpool
cannot import 'rool': I/O error. Destroy and re-create the pool from a backup source.

то восстановить пул практически невозможно. Но можно хотя бы посмотреть, что случилось с ним.

$ # zpool import -fFVXm rpool
$ # zpool status rpool
  pool: rpool
 state: FAULTED
status: One or more devices could not be used because the label is missing
        or invalid.  There are insufficient replicas for the pool to continue
        functioning.
action: Destroy and re-create the pool from
        a backup source.
   see: http://www.sun.com/msg/ZFS-8000-5E
 scan: none requested
config:

        NAME                     STATE     READ WRITE CKSUM
        rpool                    FAULTED      0     0     1  corrupted data
          raidz1-0               DEGRADED     0     0     6
            7476972440919441152  FAULTED      0     0     0  was /dev/dsk/c2t0d0s0
            c2t0d2               ONLINE       0     0     0
            c2t0d0               ONLINE       0     0     0
            c2t0d3               ONLINE       0     0     0

Хочу заметить, что любые операции по работе с пулом будут недоступны, и будет ругать:

    pool not found.

То есть данная команда лишь позволяет найти «виновника», но реально что-то сделать с пулом — нельзя.

# inline comment in file

    sed -i.bak 's/^\(SearchedText_bla_bla_bla.*\)/#\1/g' /usbkey/ssh/sshd_config

# script checks

    chmod 755 *.*
    кодировка CRLF -> LF

# Bash FAQ
If you are using the && operator just to stop at the command that fails and not continue,
you might want to surround the chunk of code with set -e and close with set +e.
that way you can remove the && and your code will most likely look cleaner.

    echo first_command
    false # it doesnt stop the execution of the script

    # surrounded commands executed in a subshell
    (
    set -e
    echo successful_command_a
    false # here stops the execution of the group
    echo successful_command_b
    set +e # actually, this is not needed here
    )

    # the script is alive here
    false # it doesnt stop the execution of the script
    echo last_command

# symlink bash
    # create symbolik link to mgit if it does not exists
    # check the existence of a symlink and that it is not broken
    # otherwise - create link
    mgit_file="/opt/src/multigit/mgit"
    mgit_link="/opt/custom/sbin/mgit"
    # -L test if link exists
    # -e "${file}" fails if the symlink exists but its target does not exist. 
    [ -L ${mgit_link} ] && [ -e ${mgit_link} ] || ln -s $mgit_file $mgit_link

# heredoc to variable

    define(){ IFS='\n' read -r -d '' ${1} || true; }
    define rsyslog_conf <<-EOF
    EOF
    echo $rsyslog_conf

# Change remote git URL

    git remote set-url origin https://github.com/alexxlabs/opt.git

if "github" is an alias entry in /usbkey/ssh/ssh_config - then:

    git remote set-url origin github:alexxlabs/opt.git

# Add submodules
    git submodule add github:bahamas10/sos.git src/sos
    git submodule add https://github.com/bahamas10/sos.git src/sos

    git submodule add github:joyent/node-hyprlofs.git src/node-hyprlofs
    git submodule add https://github.com/joyent/node-hyprlofs.git src/node-hyprlofs

    git submodule add github:alexxlabs/multigit.git src/multigit
    git submodule add https://github.com/alexxlabs/multigit.git src/multigit


# Pull git submodules after cloning project from GitHub
From the root of the repo just run:

    git submodule update --init

# example of selfdocumenting script

    DOC_REQUEST=70
    if [ "$1" = "-h"  -o "$1" = "--help" ]     # Request help.
    then                                       # Use a "cat script" . . .
    cat <<README
    List the statistics of a specified directory in tabular format.
    ---------------------------------------------------------------
    The command-line parameter gives the directory to be listed.
    If no directory specified or directory specified cannot be read,
    then list the current working directory.

    README
    exit $DOC_REQUEST
    fi


# How to resolve git stash conflict without commit?
$ git stash pop
...manually resolve conflict(s)
$ git reset
$ git stash drop

# git stash
    git stash
    git stash list
    git stash pop

## Docker

### Logfiles for docker images are stored in the zone so you need to look there:

```bash
$ cat /zones/${UUID}/logs/stdio.log
```

### You may like to login with a shell for some debugging:

```bash
$ zlogin -i ${UUID} /native/usr/vm/sbin/dockerexec /bin/sh
```

## Samba

### Create a windows share

[Join Domain fist!](http://notallmicrosoft.blogspot.com/2010/11/interoperability-between-windows-and.html)

Following will create unauthenticated windows share:

```bash
zfs create zones/myshare
zfs create -o casesensitivity=mixed -o nbmand=on zones/myshare
zfs set sharesmb=name=myshare,guestok=true zones/myshare
idmap add winname:Guest unixuser:nobody

zfs set aclmode=passthrough zones/myshare
ls -dV /zones/myshare/
chmod A+user:nobody:rwxpdDaARWcCos:fd:allow /zones/myshare

chmod A2- /zones/myshare
chmod A2- /zones/myshare
chmod A2- /zones/myshare
ls -dV /zones/myshare/
```

It is possible to add limit to IP addresses by specifying:
```
rw=@192.168.0.73:@192.168.0.254
```

above will grant read write acces to only those two ip addresses.

[CIFS troubleshooting](http://wiki.illumos.org/display/illumos/CIFS+Service+Troubleshooting)

### [Creating KVM dataset for windows](http://www.philipp.haussleiter.de/2013/07/creating-a-smartos-dataset-for-win-srv-2k12r2/)

Above describes steps to create clean windows installation save it as a dataset and import to vmadm as base machine

Quickstart:

```bash
cat > /opt/base_win2012.json << EOF
{ "brand": "kvm",
"alias": "base_win2012",
"vcpus": 2,
"autoboot": false,
"ram": 16384,
"resolvers": ["192.168.0.16"],
"disks": [{
 "boot": true,
 "model": "ide",
 "size": 71680
}],
"nics": [
 {
  "nic_tag": "admin",
  "model": "e1000",
  "ip": "dhcp",
  "gateway": "192.168.0.1",
  "primary": 1
 }
]
}
EOF

$ vmadm create -f /opt/base_win2012.json
$ cp /opt/iso/win2012.iso /zones/422f5758-4fd3-4bca-be3c-f9d44e80138f/root/
$ vmadm boot 422f5758-4fd3-4bca-be3c-f9d44e80138f order=cd,once=d cdrom=/win2012.iso,ide
$ vmadm info 422f5758-4fd3-4bca-be3c-f9d44e80138f
```

### Mount Windows Share from another computer

First enable netowrk/smb/client service:
```bash
svcadm enable -r network/smb/client
```

Then mount with username:
```bash
mount -F smbfs -o user=tester,domain=ERACENT //servername/share /mnt
```

Mounted share is accessible at /mnt

## [SMART disk check](http://cuddletech.com/blog/pivot/entry.php?id=321)

```bash
iostat -Exn
```

Additionally smartmontools can be easily installed:

```bash
pkg set-publisher -O http://scott.mathematik.uni-ulm.de/release uulm.mawi
pkg search -pr smartmontools
pkg install smartmontools
```

and

```bash
/opt/tools/sbin/smartctl -a /dev/rdsk/c1t0d0 -d sat,12
```

[voila!](https://knowledgegainedandshared.com/2016/10/27/omnios-and-getting-smartmontools-to-work/)

One liner after smartctl install:

```bash
zpool status | grep -o " c[^ ]*" | cut -c 2- | while read disk;do echo "**** STATISTICS FOR /dev/rdsk/$disk";iostat -exn |grep $disk;/opt/tools/sbin/smartctl /dev/rdsk/${disk} -A -d sat,12; done
```


### SmartOS easy iso sharing

iso files can be shared via LOFS mounts.
Let's assume directory with ISO files is /zones/iso

to make it available for the zone (or KVM):
```bash
mkdir /zones/1c5eb50e-7157-4e23-b9d0-8f24e9266936/iso
mount -F /zones/iso /zones/1c5eb50e-7157-4e23-b9d0-8f24e9266936/iso
```

and it's mounted and ready to use!

No more need to copy ISO files.

### Sort snapshots by date

to list all snapshots of zones/peer1 sorted by creation date descending:

```bash
zfs list -H -t snapshot -o name -S creation -d1 zones/peer1
```

Ascending sorting can be done by using -s instead of -S parameter.

### Persist users and change password

This is for SmartOS only - as SmartOS is using removable root partition, saving password changes and new users needs additional actions:

http://wiki.smartos.org/display/DOC/Persistent+Users+and+RBAC+in+the+Global+Zone

```bash
svcadm disable svc:/system/name-service-cache:default
svcadm disable svc:/site/persist-syscfg:default
```

add users, change passwords, etc. and:

```bash
svcadm enable svc:/site/persist-syscfg:default
svcadm enable svc:/system/name-service-cache:default
```

/usbkey/shadow does not change with above instruction!
Passwords are not persisted yet.
After following above steps you will have to:

```bash
cp /usbkey/cn-persist/etc/shadow /usbkey/
```

And you are set!

### [Cofigure NTP client](http://www.c0t0d0s0.org/archives/7453-Configuring-an-NTP-client-in-Solaris-11.html)


copy /etc/inet/ntp.client to /etc/inet/ntp.conf and edit it.

Then:
```bash
svcadm enable ntp
```

Remember, that ntpdate will not work on regular port when ntp daemon is running, so we need -u parameter: ntpdate -ub chronos

### Timezone change

Edit /etc/TIMEZONE file, change the value for TZ to desired value

```
TZ=US/Pacific        ### for PDT
TZ=Asia/Calcutta     ### for IST
TZ=Poland  ### for CET time
```

for full list, refer /usr/share/lib/zoneinfo or use tzselect command to find the right timezone name (it does not change the timezone)

Run the following command in the prompt,

```bash
rtc -z Poland      ### replace "Poland" with your valid zonename
rtc -c
```

reboot the system to effect the change and issue "date" command to verify

```bash
date
```
```
Sat May 12 12:13:56 IST 2007
```

Copied from: https://blogs.oracle.com/hariblog/entry/how_do_i_change_timezone

### [HDD HotSwap](http://serverfault.com/questions/556929/how-to-make-solaris-rescan-disk-info-after-hotswap)

You need to first initiate devfsadm cleanup subroutines.

```bash
devfsadm -C -c disk -v
```

Then, configure and create device path

```bash
devfsadm -c disk -v
```

If that is unsuccessful, then...

Remove the disk.

```bash
cfgadm -c unconfigure sata0/5::dsk/c2t5d0
```

Initiate devfsadm cleanup subroutines.

```bash
devfsadm -C -c disk -v
```

Verify the disk has been removed.

```bash
cfgadm -al
ls -ld /dev/dsk/c2t5d0*
```

Configure and create device paths

```bash
devfsadm -c -v
cfgadm -c configure sata0/5::dsk/c2t5d0
```

Verify the disk

```bash
cfgadm -al
ls -ld /dev/dsk/c2t5d0*
```

### [Best ZFS Manual](https://pthree.org/2013/01/03/zfs-administration-part-xvii-best-practices-and-caveats/)

Above is mainly bsd concerned, but most commands will work fine on SmartOS and Solaris

additionally, [adding to the domain](http://notallmicrosoft.blogspot.com/2010/11/interoperability-between-windows-and.html)

Here is a [Linux vs. SmartOS admin cheatsheet](https://docs.joyent.com/jpc/managing-a-smartos-instance/finding-your-way-around-in-smartos/the-joyent-linux-to-smartos-cheat-sheet)

### ZFS health check

To check general health of ZFS pool run:

```bash
zpool status
```

if any errors are detected, it is recommended to run scrub operation:

```bash
zpool scrub zones
```

after scrub finished check what files may be invalid:

```bash
zpool status -v
```

Then decide what to do with corrupt files.

To find a DISK serial number, run:

```bash
iostat -Exn
```

or

```bash
dmesg | less
```

and search for specific disk: (/c1t0d0) the serial number should be near the device name.

### Check space used by snaps

```bash
zfs list -rtsnapshot -Ho used tank/nfs | numfmt --from=iec | paste -sd+ - | bc | numfmt --to=iec
```

### Set autoreplace and Autoexpand properties of POOL

```bash
zpool set autoreplace=on zones
zpool set autoexpand=on zones
```
