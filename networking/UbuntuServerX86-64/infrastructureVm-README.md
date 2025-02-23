# Setup
* Create the Ubuntu server VM as outlined on the Wiki
* Ensure that your VM has x5 additional virtual network interfaces that are attached to VMnet1 and VMWare's DHCP server is disabled on VMnet1
* Setup git preferences for main branch by running: `$ git config --global init.defaultBranch main`
* Name your VM UbuntuX86-64-infrastructure and create a snapshot
* From your infrastructure VM, download the files into a sandbox directory using
  ```
  $ wget https://raw.githubusercontent.com/jhu-information-security-institute/infrastructure/main/networking/infrastructureVm-UbuntuServerX86-64.sh
  $ chmod +x infrastructureVm-UbuntuServerX86-64.sh
  $ ./infrastructureVm-UbuntuServerX86-64.sh
  ```
* Create Ubuntu server DHCP Docker container as described on [dhcpsvr](https://github.com/jhu-information-security-institute/infrastructure/tree/main/networking/dhcpsvr)
  * Please note that you only build it for this step.  Running it is performed later.
* Create Ubuntu server DNS Docker container as described on [dnssvr](https://github.com/jhu-information-security-institute/infrastructure/tree/main/networking/dnssvr)
  * Please note that you only build it for this step.  Running it is performed later.
* Change into the downloaded `infrastructureVm/config/UbuntuServerX86-64` directory
* Run the installer1 with sudo using: `$ sudo ./infrastructureVm-install1.sh`
    * Note: this step disabled NetworkManager
* Run the installer2 using: `$ ./infrastructureVm-install2.sh`
* Edit nameserver IP address and domain name in nameserver section of `ens33` portion within `/etc/netplan/99-config.yaml` to values for your internet router
  * Run `$ ip route show` in a terminal and use whatever ip address the "default via XXX.XXX.XXX.XXX" is
  * Ensure that the device names `ens??` match with those created in your vm
  * Note: only the first 3 interfaces are used (e.g., ens33, ens37, and ens38)
* Ensure that the device names `ens??` within `~/netplan/100-config.yaml` match with those created in your vm
* To eliminate the warnings shown below, run the commands below to restrict the permissions on the netplan *.yaml configuration files
  ```
  $ sudo chmod 600 /etc/netplan/99-config.yaml
  $ chmod 600 ~/netplan/100-config.yaml
  ```
  * Warnings
    ```
    ** (generate:3569): WARNING **: 14:10:11.987: Permissions for /etc/netplan/100-config.yaml are too open. Netplan configuration should NOT be accessible by others.
    ** (generate:3569): WARNING **: 14:10:11.987: Permissions for /etc/netplan/99-config.yaml are too open. Netplan configuration should NOT be accessible by others.
    ``` 
* Run `$ sudo netplan apply`
* Ensure that /etc/NetworkManager/NetworkManager.conf has the lines below
  ```
  [ifupdown]
  managed=false
  ```
* Start Ubuntu server DHCP Docker container as described on [dhcpsvr](https://github.com/jhu-information-security-institute/infrastructure/tree/main/networking/dhcpsvr)
* Start Ubuntu server DNS Docker container as described on [dnssvr](https://github.com/jhu-information-security-institute/infrastructure/tree/main/networking/dnssvr)    
* Shutdown UbuntuServerX86-64-infrastructure and take a snapshot

# Startup
* UbuntuX86-64-infrastructure should always boot first (before UbuntuX86-64-target) because it provides DHCP and DNS to VMNet1
* After booting UbuntuX86-64-infrastructure
  * Start dnssvr (uses ens38, static ip) container using `$ docker start dnssvr`
  * Start dhcpsvr (uses ens39, static ip) container using `$ docker start dhcpsvr`
  * Configure dhcp assigned addresses by running `$ sudo ~/netplan/warmstart-netplan.sh -c ~/netplan/100-config.yaml`
  * Start other containers (e.g., suricata or auth)
    * start suricata (uses ens41, static ip) container using `$ docker start suricata`

# Shutdown
* Always shut down UbuntuX86-64-target prior to shutdown of UbuntuX86-64-infrastructure
* Before shutting down UbuntuX86-64-infrastructure
    * run `$ sudo ~/netplan/prepshutdown-netplan.sh -c 100-config.yaml`
    * run `$ docker stop dnssvr dhcpsvr`
    
# Notes
* https://netplan.readthedocs.io/en/latest/netplan-yaml
