# MDT
## Introduction
This project is for using TIG stack (Telegraf, InfluxDB, Grafana) to demonstrate Cisco Model-Driven Telemetry on IOS-XR, IOS-XE and NX-OS platforms.

It will build and run three individual container and create one Docker network with Docker Compose, and after finishing the demonstration, Docker Compose will remove these three containers and the docker network it created automatically, but the Docker images which are built by Docker Compose will remain on your system for the sake of future use.

The whole project is based on three official Docker images, telegraf:1.15.1,  influxdb:1.8.1 and grafana/grafana:7.1.1. If your network environment is not so good, it is recommended that you should prepare these three Docker images in your environment before the demonstration or using any Docker image acceleration services.
## Test environment
Even though this is container, it should run anywhere, but I still want to say **It works on my environment.** :)
My lab environment is list below:
### Docker host
```
NAME="Ubuntu"
VERSION="18.04.4 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.4 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic

docker-compose version 1.26.2, build eefe0d31

Docker version 19.03.9, build 9d988398e7
```
### IOS-XR
```
RP/0/RP0/CPU0:A9Kv-1#show version
Tue Aug  4 22:50:15.431 CST
Cisco IOS XR Software, Version 7.1.1
Copyright (c) 2013-2020 by Cisco Systems, Inc.

Build Information:
 Built By     : deenayak
 Built On     : Mon Jan 27 01:20:49 PST 2020
 Built Host   : iox-ucs-033
 Workspace    : /auto/srcarchive15/prod/7.1.1/xrv9k/ws
 Version      : 7.1.1
 Location     : /opt/cisco/XR/packages/
 Label        : 7.1.1

cisco IOS-XRv 9000 () processor
System uptime is 4 weeks 7 hours 6 minutes

RP/0/RP0/CPU0:A9Kv-1#
```
### IOS-XE
```
csr-11#show version
Cisco IOS XE Software, Version 16.12.04
Cisco IOS Software [Gibraltar], Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.12.4, RELEASE SOFTWARE (fc5)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2020 by Cisco Systems, Inc.
Compiled Thu 09-Jul-20 21:51 by mcpre
...
cisco CSR1000V (VXE) processor (revision VXE) with 2080059K/3075K bytes of memory.
```
### NX-OS
```
nxosv# show version
Cisco Nexus Operating System (NX-OS) Software
TAC support: http://www.cisco.com/tac
Documents: http://www.cisco.com/en/US/products/ps9372/tsd_products_support_serie
s_home.html
Copyright (c) 2002-2020, Cisco Systems, Inc. All rights reserved.
The copyrights to certain works contained herein are owned by
other third parties and are used and distributed under license.
Some parts of this software are covered under the GNU Public
License. A copy of the license is available at
http://www.gnu.org/licenses/gpl.html.

Nexus 9000v is a demo version of the Nexus Operating System

Software
  BIOS: version
 NXOS: version 9.3(5)
  BIOS compile time:  
  NXOS image file is: bootflash:///nxos.9.3.5.bin
  NXOS compile time:  7/20/2020 20:00:00 [07/21/2020 14:30:11]


Hardware
  cisco Nexus9000 C9300v Chassis
  Intel(R) Xeon(R) Gold 6132 CPU @ 2.60GHz with 8163416 kB of memory.
  Processor Board ID 9IY0S312WAE

  Device name: nxosv
  bootflash:    4287040 kB
Kernel uptime is 1 day(s), 7 hour(s), 32 minute(s), 4 second(s)

Last reset
  Reason: Unknown
  System version:
  Service:

plugin
  Core Plugin, Ethernet Plugin

Active Package(s):

nxosv#
```
## How to use
**This is for demo purpose only, there is no persistent datastore or volume on influxdb container, so you will lost all the data in database after killing or removing the influxdb container.**
### 1. Clone this repo to your own environment
```
~$ git clone https://github.com/aoterbear/MDT.git
```
### 2. CD to the folder you just cloned
```
~$ cd ./MDT
```
### 3. Start the containers
```
~/MDT$ docker-compose up -d
```
### 4. Open the Grafana WebUI on http://localhost:3000, or change the localhost to your specific IP address, login with credential **admin/admin**
**screenshoots TBD**
### 5. Test the datastore connectivity, the InfluxDB with a database called **mdt_db** as the default datastore should been configured properly already
**screenshoots TBD**
### 6. Begin to configure your Grafana Dashboard
### 7. When you finished, stop the containers
```
~/MDT$ docker-compose down
```
### P.S Default telegraf transport protocol and listening port
```
on IOS-XR device, use TCP as protocol and port 57000
on IOS-XE device, use GRPC as protocol and port 57001
on NX-OS device, use GRPC as protocol and port 57002
```
## Device Configuration Example
**It is highly recommend that using NTP to synchronize the clock on both network devices and Docker host.**
### IOS-XR
**Streaming interfaces statistics and CPU utilization use YANG.**
```
RP/0/RP0/CPU0:A9Kv-1#sh run telemetry model-driven
Wed Aug  5 09:41:48.568 CST
telemetry model-driven
 destination-group MDT
  vrf OOB
  address-family ipv4 x.x.x.x port 57000
   encoding self-describing-gpb
   protocol tcp
  !
 !
 sensor-group cpu
  sensor-path Cisco-IOS-XR-wdsysmon-fd-oper:system-monitoring/cpu-utilization/total-cpu-one-minute
 !
 sensor-group interfaces
  sensor-path Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/data-rate
  sensor-path Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters
 !
 subscription MDT
  sensor-group-id cpu sample-interval 5000
  sensor-group-id interfaces sample-interval 5000
  destination-id MDT
 !
!

RP/0/RP0/CPU0:A9Kv-1#
```
### IOS-XE
**Enable Netconf first.**
```
netconf-yang
```
**Streaming interfaces statistics and CPU utilization use YANG.**
```
csr-11#show run | section telemetry
telemetry ietf subscription 1
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface/statistics
 source-address y.y.y.y
 source-vrf MGMT
 stream yang-push
 update-policy periodic 500
 receiver ip address x.x.x.x 57001 protocol grpc-tcp
telemetry ietf subscription 2
 encoding encode-kvgpb
 filter xpath /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds
 source-address y.y.y.y
 source-vrf MGMT
 stream yang-push
 update-policy periodic 500
 receiver ip address x.x.x.x 57001 protocol grpc-tcp
csr-11#
```
### NX-OS
**Enable telemetry and GRPC.**
```
feature telemetry
feature grpc
```
**Streaming Ethernet1/1 and Ethernet1/2 interface counters and CPU utilization using the default DME data-source.**
```
telemetry
  destination-group MDT
    ip address x.x.x.x port 57002 protocol gRPC encoding GPB
    use-vrf management
  sensor-group CPU_MDT
    data-source DME
    path sys/procsys/syscpusummary
  sensor-group Interface_MDT
    data-source DME
    path sys/intf/phys-[eth1/1]/dbgEtherStats
    path sys/intf/phys-[eth1/1]/dbgIfIn
    path sys/intf/phys-[eth1/1]/dbgIfOut
    path sys/intf/phys-[eth1/2]/dbgEtherStats
    path sys/intf/phys-[eth1/2]/dbgIfIn
    path sys/intf/phys-[eth1/2]/dbgIfOut
  subscription 1
    dst-grp MDT
    snsr-grp Interface_MDT sample-interval 5000
  subscription 2
    dst-grp MDT
    snsr-grp CPU_MDT sample-interval 5000

nxosv#
```
## Screenshoots
**TBD**
## To Do
**TBD**
## Useful Links
### Cisco MDT telegraf plugin
https://github.com/influxdata/telegraf/tree/master/plugins/inputs/cisco_telemetry_mdt
### Cisco IOS-XR MDT
https://xrdocs.io/telemetry/
### Cisco NX-OS DME
https://developer.cisco.com/site/nxapi-dme-model-reference-api/?version=9.3(5)
### ANX as the YANG Module Explorer
https://github.com/cisco-ie/anx
