# *****************************************************************************************
# ************ Setup virtual VNICs on windows desktop to support different VLANs **********
# *****************************************************************************************
#
# before we do ALL this, we need to setup hardware NIC

 -	in properties of network adapter - дополнительно
 	set 'Packet Priority & VLAN' to 'VLAN Enabled'
 -	optionally, rename NIC to 'vlan-base' as we use in our setup

inspired by: https://taeffner.net/2022/04/multiple-vlans-windows-10-11-onboard-tools-hyper-v/

Examining the advanced tab of my NIC, I've seen that setting a single VLAN ID is not hard,
it can be done in advanced properties of the NIC or using powershell. In our case,
as we need to operate on multiple VLANs things are different.
There is no option to add a virtual NIC so I started tinkering.

# The solution is Hyper-V

So after some research I found out, that Hyper-V is VLAN aware
and you can use their network stack without installing and running the full hypervisor.
Additionally you can create multiple virtual Host-NICs attached to a VLAN aware Hyper-V-Switch.

Sounds like Rocket science? It is actually quite straight-forward but you need to configure that using powershell.

## Step 1: Installing the Windows components
#
In the windows feature installation dialog, we have to install the two components:
 - Hyper-V-Services
 - Hyper-V-Module for Windows powershell

After the setup is done, you have to reboot the computer.

## Step 2: Setting up the vSwitch
#
Hyper-V automatically creates a 'Default vSwitch' that you can not delete and unfortunately it is not VLAN-Aware.
We will have to keep this one as it is and go ahead creating a second vSwitch.

For this, we need to open up powershell as an administrator and enter the following commands.
( Your host will lose it's network connection – Do not do that remotely or with apps running. )

$ Get-NetAdapter
$
This will return a list of network adapters, find your physical NIC and note its "Name".
In most cases "Ethernet", but we rename it to 'vlan-base'

$ New-VMSwitch -Name "vEthernet" -NetAdapterName vlan-base -AllowManagementOS:$true
$
This command creates a new virtual switch with the name vEthernet.
The -NetAdapterName parameter is used to specify the physical network adapter to bind.
The -AllowManagementOS parameter with a $true value is used to allow the host OS to share
the virtual switch and physical NIC with the VMs.

to view ALL switches: Get-VMSwitch
to delete it: Remove-VMSwitch "vEthernet"

We do now have a clean new VLAN-Aware vSwitch

## Step 3: Setting up VLAN interfaces
#
# Now we create a new virtual Host-NIC and assign a VLAN tag 123 to it.
# Please note, that the interface name can be chosen freely.
# One might want to name them by purpose.

$ Add-VMNetworkAdapter -ManagementOS -Name "VLAN33" -SwitchName "vEthernet" -Passthru | Set-VMNetworkAdapterVlan -Access -VlanId 33
$ Remove-VMNetworkAdapter -ManagementOS -Name VLAN33

$ Add-VMNetworkAdapter -ManagementOS -Name "VLAN9" -SwitchName "vEthernet" -Passthru | Set-VMNetworkAdapterVlan -Access -VlanId 9
$ Remove-VMNetworkAdapter -ManagementOS -Name VLAN9

$ Add-VMNetworkAdapter -ManagementOS -Name "VLAN8" -SwitchName "vEthernet" -Passthru | Set-VMNetworkAdapterVlan -Access -VlanId 8
$ Remove-VMNetworkAdapter -ManagementOS -Name VLAN8

# Finally, verify that all adapter are in place
#
$ Get-NetAdapter