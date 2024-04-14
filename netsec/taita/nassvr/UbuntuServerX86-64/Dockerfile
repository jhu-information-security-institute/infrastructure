# Description:
#   This runtime environment example Dockerfile creates a container with a minimal Ubuntu server running nfs and tftp servers 
# Usage:
#   From this directory, run $ docker build -t tnassvr .
#   By default, runs as root
# https://help.ubuntu.com/community/SettingUpNFSHowTo
# https://help.ubuntu.com/community/NFSv4Howto

FROM ubuntu:22.04

#https://grigorkh.medium.com/fix-tzdata-hangs-docker-image-build-cdb52cc3360d
ENV TZ=US/Eastern
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

#avoid question/dialog during apt-get installs
ARG DEBIAN_FRONTEND noninteractive

# Setup container's ENV for systemd
ENV container docker

#update
RUN apt-get update

#install utilities
RUN apt-get install apt-utils dpkg-dev debconf debconf-utils -y
RUN apt-get install net-tools iputils-ping iptables iproute2 wget nmap bind9-dnsutils dnsutils inetutils-traceroute isc-dhcp-common -y
RUN apt-get install vim acl sudo telnet ssh netcat nfs-common -y

#install dependencies for systemd and syslog
RUN apt-get install systemd systemd-sysv syslog-ng syslog-ng-core syslog-ng-mod-sql syslog-ng-mod-mongodb -y

#start systemd
CMD ["/usr/lib/systemd/systemd", "--system"]

#install all the things (nfs server, tftp server)
RUN apt-get install nfs-kernel-server -y
RUN apt-get install tftpd-hpa -y

#update configuration files
COPY etc_exports /etc/exports
COPY etc_default_tftpd-hpa /etc/default/tftpd-hpa

#make the directories for the shared volumes
RUN mkdir -p /netsec/home
RUN mkdir /netsec/rootfs
RUN mkdir /tftpboot

# Finished!
RUN echo $'\n\
* On the VM hosting this nfs server, make sure that you have created the /netsec/home, /netsec/rootfs, and /tftpboot directories on the VM itself prior to running the container \n\    
* Container is ready, run it using $ docker run -d --name nassvr --privileged --security-opt seccomp=unconfined --cgroup-parent=docker.slice --cgroupns private --tmpfs /tmp --tmpfs /run --tmpfs /run/lock -v /netsec/home:/netsec/home:rw -v /netsec/rootfs:/netsec/rootfs:rw -v /tftpboot:/tftpboot:rw --network host tnassvr:latest \n\
* Attach to it using $ docker exec -it nassvr bash \n\
* nfs-kernel-server status is available using # systemctl status nfs-kernel-server \n\
* nfs-kernel-server can be stopped and started as well using systemctl'
