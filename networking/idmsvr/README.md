# Application overview
## freeipa-server
* setup DNS and the hostname in the host
* (do not perform this step at present)-build the container as instructed below
* setup chrony server (ntp)
   ```
   # apt-get install chrony
   ```
•	Allow hosts on netsec.isi.jhu.edu to reach chrony by adding the following to `/etc/chrony.conf`
   ```
   allow 192.168.25.0/24
   ```
* configure ipa server: `# ipa-server-install --hostname=auth.netsec-docker.isi.jhu.edu --domain=netsec-docker.isi.jhu.edu --realm=NETSEC-DOCKER.ISI.JHU.EDU`
    * say no when asked to configure integrated DNS
    * specify passwords when prompted
* Configure the firewall in the host (we are opening ports this host is running servers from)
    ```
    # firewall-cmd --add-service={freeipa-ldap,freeipa-ldaps,ntp,kerberos}
    # firewall-cmd --runtime-to-permanent
    ```
* Test it is working by logging in
    ```
    # kinit admin
    # ipa user-find admin
    ```
## clients
* setup DNS and the hostname
* setup chrony (ntp)
    * install it: `$ sudo apt-get install chrony`
    * Add the following line to `/etc/chrony/chrony.conf`
    ```
    pool ipa.netsec.isi.jhu.edu iburst
    ```
    * Then, enable, reload, start chrony service and query status with:
    ```
    $ sudo systemctl daemon-reload
    $ sudo systemctl enable chrony
    $ sudo systemctl restart chrony
    ```
* check chrony is working using: `$ chronyc sources`
* install the freeipa-client package
    ```
    $ sudo apt-get install freeipa-client
    ```
    * realm=NETSEC-DOCKER.ISI.JHU.EDU
    * kerberos server=auth.netsec-docker.isi.jhu.edu
    * administrative server=auth.netsec-docker.isi.jhu.edu
* run the client installer: `# ipa-client-install`
* After installing freeipa-client in ubuntu, sssd is configured to activate certain
services in /etc/sssd/sssd.conf (source):
    ```
    ...
    [sssd]
    services = nss, pam, ssh, sudo
    ...
    ```
* Disable the `sssd-*.socket` socket-activated systemd services (note, these are enabled by default) to eliminate receiving errors in the journald log when booting
    ```
    # systemctl disable sssd-nss.socket
    # systemctl disable sssd-pam.socket
    # systemctl disable sssd-pam-priv.socket
    # systemctl disable sssd-sudo.socket
    # systemctl disable sssd-ssh.socket
    ```

## CentOS
The freeipa server instances on CentOS run in a single docker container.

# Runtime environment setup

**!!!!!!!!!!!!!!!!!!!!!!!!!Below is no longer working.  Do not use Docker with this part of the assignment right now!!!!!!!!!!!!!!!!!!!!!!**

## CentOS
1. Download files to build container
    ```
    $ wget https://raw.githubusercontent.com/jhu-information-security-institute/infrastructure/main/networking/idmsvr/idmsvr_CentOsX86-64.sh
    $ chmod +x idmsvr_CentOsX86-64.sh
    $ ./idmsvr_CentOsX86-64.sh
    ```
1. Build, run, attach to container
    ```
    $ docker build -t tidmsvr .
    $ docker run -d --name idmsvr --hostname auth.netsec-docker.isi.jhu.edu --privileged --network host --cpus=2 tidmsvr:latest
    $ sudo firewall-cmd --add-service={freeipa-ldap,freeipa-ldaps,dns,ntp,kerberos}
    $ sudo firewall-cmd --runtime-to-permanent
    $ docker exec -it idmsvr bash
    ```
1. Install the server (run in the container)
    ```
    # ipa-server-install --hostname='auth.netsec-docker.isi.jhu.edu' --domain=netsec-docker.isi.jhu.edu --realm=NETSEC-DOCKER.ISI.JHU.EDU --no-ntp
    ```
# Useful websites
* https://www.freeipa.org/page/Quick_Start_Guide

