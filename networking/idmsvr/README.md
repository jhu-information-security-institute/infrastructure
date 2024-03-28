**Do not use as these instructions are still in work!!**








# Application overview
## freeipa-server

## CentOS
The freeipa server instances on CentOS run in a single docker container.

# Runtime environment setup
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

