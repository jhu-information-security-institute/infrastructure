# Runtime environment setup
## Server
1. Create x3 disks for the VM with ext4 logical volumes
    * /netsec/rootfs for the RPI rootfs images
    * /tftpboot for the RPI PXE boot images
    * /netsec/home for the user home directories
    * format the logical volumes
    * create the directories for the mounts below
1. Prep the volumes
    * Update `/etc/fstab` in the VM (see example commands in [etc_fstab](https://github.com/jhu-information-security-institute/infrastructure/raw/main/netsec/taita/nassvr/UbuntuServerX86-64/etc_fstab)) to mount the x3 logical volumes created above using their appropriate `/dev/sd<X>1` into `/mnt/nfs_home`, `/mnt/nfs_rootfs`, `/mnt/tftp`
    * Update `/etc/fstab` in the VM to create bind mounts between `/mnt/nfs_home` <-> `/netsec/home`, `/mnt/nfs_rootfs` <-> `/netsec/rootfs`, and `/mnt/tftp` <-> `/tftpboot`
    * Reboot the VM
1. Disable the firewall on the appropriate port:
    `$ sudo ufw allow 2049/tcp`
1. Download files
    ```
    $ wget https://raw.githubusercontent.com/jhu-information-security-institute/infrastructure/main/netsec/taita/nassvr/nassvr_UbuntuServerX86-64.sh
    $ chmod +x nassvr_UbuntuServerX86.sh
    $ ./nassvr_UbuntuServerX86.sh
    ```
1. Install packages
   ```
   $ sudo apt-get install nfs-kernel-server -y
   $ sudo apt-get install tftpd-hpa -y
   ```
1. Update `/etc/exports` (see [etc_exports](https://github.com/jhu-information-security-institute/infrastructure/raw/main/netsec/taita/nassvr/UbuntuServerX86-64/etc_exports)) and `/etc/default/tftpd-hpa` (see [etc_default_tftpd-hpa](https://github.com/jhu-information-security-institute/infrastructure/raw/main/netsec/taita/nassvr/UbuntuServerX86-64/etc_default_tftpd-hpa))
1. Ensure `/etc/hosts.allow` contains
    ```
    ALL:ALL
    ```
1. Export the shares: `# exportfs -ra`
1. Start the nfs server: `# systemctl start nfs-kernel-server`
1. Make a file to see from the client: `# touch /netsec/rootfs/hello`
1. View the current exports: `# showmount -e`
