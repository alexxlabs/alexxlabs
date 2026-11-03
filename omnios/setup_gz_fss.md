# Changing the system default scheduler

If you plan to run a number of zones on an OmniOS system, consider changing the system scheduler to the
Fair-Share-Scheduler (FSS) so that zones get an equal share of system resources.
See the FSS(7) manual page for more details: https://illumos.org/man/4/FSS

The following command sets FSS as the default scheduler for the system:

	omnios# dispadmin -d FSS

By default each zone will get an equal share of system resources;
each zone has a cpu-shares value of 1.

This value can be adjusted on a per-zone basis to assign relative priority to zones.

The following will give the test zone twice the priority of all others.

	omnios# zonecfg -z test set cpu-shares=2

### Set cpu-shares for GlobalZone:

	omnios# zadm set global cpu-shares=14

