# Application overview
Suricata is the leading independent open source threat detection engine. By combining intrusion detection (IDS), intrusion prevention (IPS), network security monitoring (NSM) and PCAP processing, Suricata can quickly identify, stop, and assess even the most sophisticated attacks.

## UbuntuServerX86-64-infrastructure VM
The Suricata instance on Ubuntu runs in a Docker container in our main infrastructure VM.

# Runtime environment setup

## Ubuntu
1. Download files to build container
    ```
    $ wget https://raw.githubusercontent.com/jhu-information-security-institute/infrastructure/main/networking/suricata/suricata-UbuntuServerX86-64.sh
    $ chmod +x suricata-UbuntuServerX86-64.sh
    $ ./suricata-UbuntuServerX86-64.sh
    ```
1. Build, run, attach to container
    ```
    $ docker build -t tsuricata .
    $ docker run -d --name suricata --hostname suricata.netsec-docker.isi.jhu.edu --add-host suricata.netsec-docker.isi.jhu.edu:127.0.1.1 --dns 192.168.25.10 --dns-search netsec-docker.isi.jhu.edu --privileged --security-opt seccomp=unconfined --cgroup-parent=docker.slice --cgroupns private --tmpfs /tmp --tmpfs /run --tmpfs /run/lock --network host --cpus=1 tsuricata:latest   
    $ docker exec -it suricata bash 
    ```
1. Review the settings in `/etc/default/suricata`
    * Inspect the `IFACE` setting (note: this setting must match your interface name for the interface associated with `ids.netsec-docker.isi.jhu.edu`)
1. Review the settings in /etc/suricata/suricata.yaml
    * Inspect the `HOME_NET` and `EXTERNEL_NET` settings
    * Inspect the `interface` setting in `af-packet` section (note: this setting must match your interface name for the interface associated with `ids.netsec-docker.isi.jhu.edu`)
    * Inspect the files listed in the `rule-files` section    
1. Edit `/var/lib/suricata/rules/test-ping.rules` and update IP addresseses appropriately
1. Reload rules
1. Restart Suricata

# Notes
* Signature files located in `/var/lib/suricata/rules`
* Alerts located in `/var/log/suricata` directory
* Reload rules using `$ suricata-update`
* Restart Suricata service using `$ sudo systemctl restart suricata`

# Useful websites
* https://suricata.io
