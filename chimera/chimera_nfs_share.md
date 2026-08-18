################################## ON SERVER ######################################

apk add nfs-utils-server nfs-utils-server-dinit

nano /etc/exports

# Share /data with read-write access to a specific subnet
# /path/to/share   client_IP_or_subnet(options)
/opt 10.2.0.0/24(rw,sync,no_subtree_check,no_root_squash)

	rw: Grants the client both read and write access to the directory.
	ro: Restricts the client to read-only access (default behavior).
	sync: Forces the server to write changes to disk before responding to the client request (safest option).
	no_subtree_check: Disables verification of subdirectories, improving transfer speeds and reliability.
	no_root_squash: Allows a remote root user to retain root privileges on the shared folder (use with caution)

# Enable the service to launch at boot
dinitctl enable nfs-server

# Start the service immediately
dinitctl start nfs-server

NOTE: Starting 'nfs-server' service automatically pulls in required dependencies like rpcbind and rpc.mountd.

Apply configuration changes:			exportfs -ra
View all active exports (verbose mode):	exportfs -v

################################## ON CLIENT ######################################

$ apk add nfs-utils

# adjust /etc/fstab, adding:

#	<file system> <mount point>   <type>  <options>                   <dump>  <pass>
#	10.2.0.12:/opt  /opt   nfs     defaults,_netdev,nofail     0       0
10.2.0.12:/opt  /opt    nfs     ro,hard,rsize=65536,wsize=65536,noatime,_netdev,nofail  0       0

	rw				Read-write access
	hard			Keep retrying if server is unavailable
	rsize=65536		64 KB read buffer for better throughput
	wsize=65536		64 KB write buffer for better throughput
	noatime			Skip access time updates for performance
	_netdev			Treat the mount as network-dependent
	nofail			Allows the system to finish booting successfully even if the remote
					NFS server cannot be reached at startup.

after this, run 'mount -a' or 'mount /opt' to mount given share from server