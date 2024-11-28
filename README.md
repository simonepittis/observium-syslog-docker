# Docker container for Observium Community Edition
Observium is network monitoring with intuition. It is a low-maintenance auto-discovering network monitoring platform supporting a wide range of device types, platforms and operating systems including Cisco, Windows, Linux, HP, Juniper, Dell, FreeBSD, Brocade, Netscaler, NetApp and many more. Observium focuses on providing a beautiful and powerful yet simple and intuitive interface to the health and status of your network. For more information, go to http://www.observium.org site.

Available platform are:-
* AMD64 (Intel x86_64)

### Use Docker Composer
- Follow instuctions below to create extra working directory of docker containers.
```
  $ mkdir /home/docker/observium
  $ cd observium
  $ mkdir db lock mysql
```
> You can change /home/docker directory to your desired directory and you need to change the volume mapping directories in docker-compose.yml file also.

- Download docker-compose.yml file from https://github.com/simonepittis/observium-syslog-docker github repository. Then, place docker-compose.yml file into /home/docker/observium directory.

- Run both database and observium containers.
```
  $ docker-compose up
```

## Changes
- Add Syslog-NG enable on Observium image.

## Source Repository
See source of project at https://github.com/simonepittis/observium-syslog-docker
