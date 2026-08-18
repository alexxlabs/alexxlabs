### enable zfs native SMB shares

# https://www.tumfatig.net/2023/smb-shares-using-omnios-zones-and-zfs/

#
The smbd(8) daemon manages CIFS/SMB requests. Start the daemon using SMF.
# svcadm enable -r smb/server
# svcs -d smb/server
STATE          STIME    FMRI
online         23:05:58 svc:/milestone/network:default
online         23:06:00 svc:/system/filesystem/local:default
online         23:30:59 svc:/system/idmap:default
online         23:30:59 svc:/network/smb/client:default
# sharectl status
smbfs   online client
autofs  online client
nfs     disabled
smb     online

Configure global options using sharectl(8) and smb(5) properties.

# sharectl set -p system_comment="OmniOS SMB/CIFS Service" smb
# sharectl set -p netbios_enable=true smb

# sharectl get smb
system_comment=OmniOS SMB/CIFS Service
max_workers=1024
netbios_enable=true
(...)

# sharectl get -p min_protocol -p max_protocol -p encrypt smb
min_protocol=
max_protocol=
encrypt=disabled

Announce the SMB Service
Multicast DNS and DNS Service Discovery allows LAN machines to discover the SMB shares. The service has to be enabled.

# svcadm enable dns/multicast

Manage Groups and Users
The SMB service can either join a workgroup or an Active Directory domain. At $HOME, I only use the default WORKGROUP.

The service uses SMB local groups and users. Password management using passwd is enabled through PAM.
# echo "other password required pam_smb_passwd.so.1" >> /etc/pam.conf
# passwd alexxlabs
# smbadm enable-user alexxlabs
#

Public SMB share
The defined privacy policy is:

A single user is the owner and has full management powers on the data.
Any authenticated user can read-only the data.
Any unauthenticated user (guest) can read-only the data.
Configure and expose the share using sharemgr(8) properties.

# zfs set sharesmb=name=OmniOS_ISO,guestok=true zones/iso
# sharemgr show -pv

