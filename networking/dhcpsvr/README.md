# Application overview
Enterprise-grade solution for IP address-configuration needs

ISC DHCP offers a complete open source solution for implementing DHCP servers, relay agents, and clients. ISC DHCP supports both IPv4 and IPv6, and is suitable for use in high-volume and high-reliability applications.

# Runtime environment setup
1. Download files to build container into a sandbox directory using
    ```
    $ wget https://raw.githubusercontent.com/jhu-information-security-institute/infrastructure/main/networking/dhcpsvr/dhcpsvr-UbuntuServerX86-64.sh
    $ chmod +x dhcpsvr-UbuntuServerX86-64.sh
    $ ./dhcpsvr-UbuntuServerX86-64.sh
    ```
1. Build, run, attach to container
    ```
    $ docker build -t tdhcpsvr .
    $ docker run -d --name dhcpsvr --hostname dhcp.netsec-docker.isi.jhu.edu --add-host dhcp.netsec-docker.isi.jhu.edu:127.0.1.1 --dns 192.168.25.10 --dns-search netsec-docker.isi.jhu.edu --privileged --security-opt seccomp=unconfined --cgroup-parent=docker.slice --cgroupns private --tmpfs /tmp --tmpfs /run --tmpfs /run/lock --network host --cpus=1 tdhcpsvr:latest
    $ docker exec -it dhcpsvr bash 
    ```
1. Enable the server using: `(container) # systemctl enable isc-dhcp-server`
1. Edit `/etc/dhcp/dhcpd.conf` and `/etc/default/isc-dhcp-server` to update MAC addresseses appropriately to match your virtual network adapters
    * You can view MAC addresses using `$ ip link show`
1. Reload the configuration by running `(container) # systemctl daemon-reload`
1. To eliminate the error mentioned below, disable isc-dhcp-server6 by running `(container) # systemctl disable isc-dhcp-server6`
    * Error message: "Can't create PID file /run/dhcp-server/dhcpd.pid: No such file or directory."
1. Restart the server using: `(container) # systemctl restart isc-dhcp-server`

## Notes
* Restart the server using: `(container) # systemctl restart isc-dhcp-server`
* Check the server status (there should be no errors) using: `(container) # systemctl status isc-dhcp-server`
* View the server log: `(container) # journalctl -u isc-dhcp-server`
* Configure the server by editing `/etc/dhcp/dhcpd.conf`

# Useful websites
* https://www.isc.org/dhcp
* https://linux.die.net/man/5/dhcpd-options
* https://www.iana.org/assignments/bootp-dhcp-parameters/bootp-dhcp-parameters.xhtml
