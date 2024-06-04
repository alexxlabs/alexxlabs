sbdadm create-lu /dev/zvol/rdsk/zones/iscsi_win_alexxdesk
Created the following LU:

              GUID                    DATA SIZE           SOURCE
--------------------------------  -------------------  ----------------
600144f07aabba4600006629f15f0001  107374182400         /dev/zvol/rdsk/zones/iscsi_win_alexxdesk

stmfadm add-view 600144f07aabba4600006629f15f0001 # uid from previous command
itadm create-tpg alexxlabs 10.2.0.10

itadm create-target -t alexxlabs  -n iqn.2010-08.org.illumos:02:iscsi-win-alexxdesk-100T
Target iqn.2010-08.org.illumos:02:iscsi-win-alexxdesk-100t successfully created


Windows 10 - 11 installation on a iSCSI target: https://github.com/ipxe/ipxe/discussions/324

## inspired by http://superuser.com/questions/386506/hosting-iscsi-on-smartos
### Creating an ISCSI Target

Install iscsi target (OmniOS)
```bash
pkg install pkg:/network/iscsi/target
```

enable the storage server and iscsi target server if necessary

```bash
svcadm enable stmf
svcadm enable -r svc:/network/iscsi/target:default
```

create a volume if necessary (sparse 100GB in example)

```bash
zfs create -V 100G -s zones/iscsi
```

create a logical unit

```bash
sbdadm create-lu /dev/zvol/rdsk/zones/iscsi
```

Add a view on it (GUID is output by previous command or list-lu)

```bash
stmfadm add-view GUID
```

Create a target group to connect to (Choose a GROUPNAME and a TARGETNAME and use an IP from the current server) An Example TARGETNAME could be: iqn.2010-08.org.illumos:02:iscsi-100T where the iscsi-100T part can be whatever you like. I thing creating the target group is not essential, though it might be sensible.

```bash
itadm create-tpg GROUPNAME IP
```

Create a target in this group

```bash
itadm create-target -t GROUPNAME  -n TARGETNAME
```

BTW: If you dont set a TARGETNAME a unique identifier will be generated. But this happens everytime on boot on smartos and thus clients wont be able to reconnect automatically.

