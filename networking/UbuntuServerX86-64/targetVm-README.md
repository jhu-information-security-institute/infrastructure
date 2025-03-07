# Initial setup
* Clone the Ubuntu server VM as outlined on the Wiki
* Ensure that your VM has x4 additional virtual network interfaces that are attached to VMnet1 and VMWare's DHCP server is disabled on VMnet1
* Setup git preferences for main branch by running: `$ git config --global init.defaultBranch main`
* Name your VM UbuntuServerX86-64-target and create a snapshot
* From your target VM, download the files into a sandbox directory using
   ```
    $ wget https://raw.githubusercontent.com/jhu-information-security-institute/infrastructure/main/networking/targetVm-UbuntuServerX86-64.sh
    $ chmod +x targetVm-UbuntuServerX86-64.sh
    $ ./targetVm-UbuntuServerX86-64.sh
    ```
* Change into the downloaded `targetVm/config/UbuntuServerX86-64` directory
* Run the installer using: `$ sudo ./targetVm-install.sh`
    * Note: this step disabled NetworkManager
* Edit nameserver IP address and domain name in nameserver section of `ens33` portion within `/etc/netplan/101-config.yaml` to values for your internet router
  * Ensure that the device names `ens??` within `/etc/netplan/101-config.yaml` match with those created in your vm
* Ensure that /etc/NetworkManager/NetworkManager.conf has the lines below
    ```
    [ifupdown]
    managed=false
    ```
* Run the commands below to restrict the permissions on the netplan *.yaml configuration files
  ```
  $ sudo chmod 600 /etc/netplan/101-config.yaml
  ```
  
# Final setup
* Use `$ ip link show` to query the ethernet mac addresses the VMNet1 virtual network adapters on UbuntuX86-64-target
* Similar to the step for the infrastructure VM, update `/etc/dhcp/dhcpd.conf` and `/etc/default/isc-dhcp-server` in the dhcpsvr container (on the infrastructure VM) based on your ethernet mac addresses from above
* Reload and restart isc-server-dhcp in the infrastructure VM's container
* Shutdown UbuntuServerX86-64-target and take a snapshot

# Startup
* Always boot UbuntuX86-64-target after booting UbuntuServerX86-64-infrastructure VM
* Start desired containers (e.g., nginxsvr or mailsvr) using `$ docker start <containerName>`

# Shutdown
* Always shut down UbuntuX86-64-target prior to UbuntuX86-64-infrastructure
    
# Notes
* If you lose dns for internet sites, it is likely that your `/etc/resolv.conf` needs updating
  * In your VM, add your internet router's IP address as an additional nameserver by creating a new line in `/etc/resolv.conf`
  ```
  nameserver <IPADDRESSOFINTERNETROUTER>
  nameserver 8.8.8.8
  ```
