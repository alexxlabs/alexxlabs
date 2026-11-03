### https://omnios.org/upgrade

Performing the Upgrade

The upgrade process creates a clone of the current boot environment in order to upgrade it.
Any changes made to the current boot environment after the update is started will be lost
once you reboot into the new one. This includes changes to log files so you may wish to disable
services that produce data or log entries that you don’t want to lose before starting.

Make sure the global zone can reach the network.

Create a backup boot environment for safety:

# beadm create <appropriate-backup-name>

Change the publisher in the global zone and all linked zones
(e.g. lipkg and sparse branded zones). For example, going to r151058:

# pkg set-publisher -r -O https://pkg.omnios.org/r151058/core omnios
# pkg set-publisher -r -O https://pkg.omnios.org/r151058/extra extra.omnios

If you have any ipkg branded zones, detach them following the instructions below.

Perform the update, optionally specifying the new boot-environment name:

# pkg update -f -r --be-name=r151058

This will create a new BE and install the new packages into it. When this is complete,
reboot your system. The new BE will now be the default option in loader.

Reboot

# init 6

If applicable, re-attach any ipkg zones:

# zoneadm -z <zonename> attach -u

### ipkg-branded zones only - detach procedure

For ipkg branded zones only, it is necessary to shut down and detach prior to upgrading:

  # zoneadm -z <zonename> shutdown
  ... use the following command to check when the zone has shut down ...
  # zoneadm -z <zonename> list -v
  # zoneadm -z <zonename> detach

It is also a good idea to take a ZFS snapshot of the zone root in case it’s needed for rollback
(such as if there are issues with the zone upgrade.)

  # zfs snapshot -r /path/to/zone@<old-release>

### Troubleshooting

In case a boot environment (BE) is not updating to the latest release, try to enable an older BE (in the example below omnios-r151044) and reboot. Then, update the BE. Make sure to use a new BE for the update.

For example:

# beadm activate omnios-r151044
# init 6
# pkg update -f -r --be-name=r151058

### Installing the OmniOS CA Certificate

If upgrading from OmniTI OmniOS first install the OmniOS CA certificate:

# wget -P /etc/ssl/pkg https://downloads.omnios.org/ssl/omniosce-ca.cert.pem
# openssl x509 -fingerprint -in /etc/ssl/pkg/omniosce-ca.cert.pem -noout

SHA1 Fingerprint=8D:CD:F9:D0:76:CD:AF:C1:62:AF:89:51:AF:8A:0E:35:24:4C:66:6D