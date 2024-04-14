# Runtime environment setup
## Server
1. Create x3 disks for the VM with ext4 logical volumes
    * /netsec/rootfs for the RPI rootfs images
    * /tftpboot for the RPI PXE boot images
    * /netsec/home for the user home directories
1. Prep the volumes
    * Update `/etc/fstab` in the VM (see example commands in ) to mount the x3 logical volumes created above using their appropriate `/dev/sd<X>1` into `/mnt/nfs_home`, `/mnt/nfs_rootfs`, `/mnt/tftp`
    * Update `/etc/fstab` in the VM to create bind mounts between `/mnt/nfs_home` <-> `/netsec/home`, `/mnt/nfs_rootfs` <-> `/netsec/rootfs`, and `/mnt/tftp` <-> `/tftpboot`
    * Reboot the VM
1. Disable the firewall on the appropriate port:
    `$ sudo ufw allow 2049/tcp`
1. Download files to build container
    ```
    $ wget https://raw.githubusercontent.com/jhu-information-security-institute/infrastructure/main/netsec/taita/nassvr/nassvr_UbuntuServerX86-64.sh
    $ chmod +x nassvr_UbuntuServerX86.sh
    $ ./nassvr_UbuntuServerX86.sh
    ```
1. Build, run, attach to container
    ```
    $ docker build -t tnassvr .
    $ docker run -d --name nassvr --privileged --security-opt seccomp=unconfined --cgroup-parent=docker.slice --cgroupns private --tmpfs /tmp --tmpfs /run --tmpfs /run/lock -v /netsec/home:/netsec/home:rw -v /netsec/rootfs:/netsec/rootfs:rw -v /tftpboot:/tftpboot:rw --network host tnassvr:latest
    $ docker exec -it nassvr bash 
    ```
1. From inside the docker session:
    * Ensure `/etc/hosts.allow` contains
    ```
    ALL:ALL
    ```
    * Export the shares: `# exportfs -a`
    * Restart the nfs server: `# systemctl start nfs-kernel-server`
    * Make a file to see from the client: `# touch /netsec/rootfs/hello`
