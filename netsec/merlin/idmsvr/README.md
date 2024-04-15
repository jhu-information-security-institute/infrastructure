1. Download files
    ```
    $ wget https://github.com/jhu-information-security-institute/infrastructure/raw/main/netsec/merlin/idmsvr/idmsvr_CentOsX86-64.sh
    $ chmod +x idmsvr_CentOsX86-64.sh
    $ ./idmsvr_CentOsX86-64.sh
    ```
# Application overview
## freeipa-server
* setup DNS and the hostname in the VM
* install the packages
```
# yum install krb5-workstation krb5-libs -y
# yum install freeipa-server -y
# yum install chrony -y
```
* edit `/etc/chrony.conf` and add line
```
allow 172.16.0.0/24
```
* reload and start chronyd
```
# systemctl start chronyd
# systemctl enable chronyd
```
* configure ipa server: `# ipa-server-install --hostname=merlin.netsec.isi.jhu.edu --domain=netsec.isi.jhu.edu --realm=NETSEC.ISI.JHU.EDU`
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
    pool merlin.netsec.isi.jhu.edu iburst
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
# Useful websites
* https://www.freeipa.org/page/Quick_Start_Guide

