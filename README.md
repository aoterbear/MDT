# MDT

## Environment

Even though this is container, it should run anywhere, but I still want to say **It works on my environment.** :)

My environment is below:

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

## How To Use?

1. Use git clone to copy this project on your own laptop
2. Make sure the folder you copied is called MDT, otherwise, change the name, cd to this folder
3. Excute cmd `docker-compose up -d`
4. Open Grafana UI with  http://localhost:3000, or change the "localhost" to your own specific IP address
5. The datastore is already been added, so, test the datastore connectivity first and then you can begin to configure the Grafana Dashboard

## Configuration on IOS-XR

Below is configuration on IOS-XRv 9000, version 7.1.1, this configuration works with this project default telegraf config file located in `./telegraf/data/new.conf`, you need to configure all the configuration below on your own devices manullay. This configuration should workd on all IOS-XR 7.1.1 platform.

The configuration feed device cpu utiliazatiion and interface counters as well as interface data rate only by now. You can add more sensor-group configuration to make the device to feed more data, and please remeber once you add sensor-group, the telegraf config file need be updated also. About the telegraf config file, you can refer this [link](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/cisco_telemetry_mdt).

```
telemetry model-driven
 destination-group DG_MDT
  vrf OOB
#Change x.x.x.x to your own specific IP address, it should be your docker host IP address.
  address-family ipv4 x.x.x.x port 57000  
   encoding self-describing-gpb
   protocol tcp
 sensor-group cpu
  sensor-path Cisco-IOS-XR-wdsysmon-fd-oper:system-monitoring/cpu-utilization/total-cpu-one-minute
 sensor-group interfaces
  sensor-path Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/data-rate
  sensor-path Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters
 subscription Sub_MDT
  sensor-group-id cpu sample-interval 5000
  sensor-group-id interfaces sample-interval 5000
  destination-id DG_MDT
```

## Screenshoots

TBD

## About Cisco IOS-XR Model-Driven Telemetry

Please refer this [link](https://xrdocs.io/telemetry/) for more information about Cisco IOS-XR MDT.



