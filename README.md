# IOS-XR MDT

## Introduction

This project is for using TIG stack (Telegraf, InfluxDB, Grafana) to demonstrate Cisco IOS-XR Model-Driven Telemetry.

It will setup three individual container process and create one Docker network with Docker Compose, and after you finish, Docker Compose will remove these three container process and the docker network it created automatically, but the Docker images will remain on your system for the sake of future use.

This project is based on official Docker images, telegraf:1.15.1,  influxdb:1.8.1 and grafana/grafana:7.1.1.

## Test Environment

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

**Make sure you have configured the devices properly follow the Configuration on IOS-XR chapeter first**

**This is for demo purpose only, there is no persistent datastore or volume on influxdb container, so you will lost all the data in databaseafter killing or removing the influxdb container, but you can still access the data with json format file which is ./telegraf/data/telegraf.txt**

1. Use git clone to copy this project on your own laptop
2. Make sure the folder you copied is called MDT, otherwise, change the name, cd to this folder
3. Excute cmd `docker-compose up -d` to strat all the threee containers
4. Open Grafana UI with  http://localhost:3000, or change the "localhost" to your own Docker host IP address, the default username and password are both **admin**
5. The datastore is already been added in Grafana, so, test the datastore connectivity first and then you can begin to configure the Grafana Dashboard
6. Excute cmd `docker-compose down` to stop all the three containers

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

### IOS-XR show telemetry model-driven subscription

```
RP/0/RP0/CPU0:A9Kv-1#sh telemetry model-driven subscription
Sat Aug  1 22:07:30.043 CST
Subscription:  MDT                      State: ACTIVE
-------------
  Sensor groups:
  Id                               Interval(ms)        State     
  cpu                              5000                Resolved  
  interfaces                       5000                Resolved  

  Destination Groups:
  Id                 Encoding            Transport   State   Port    Vrf     IP            
  MDT                self-describing-gpb tcp         Active  57000   OOB     172.21.2.250  
    No TLS            
```



### ./telegraf/data/telegraf.txt sample outputs

#### data_rate

```
{"fields":{"bandwidth":10000000,"input_data_rate":56,"input_load":0,"input_packet_rate":9,"load_interval":9,"output_data_rate":56,"output_load":0,"output_packet_rate":9,"peak_input_data_rate":0,"peak_input_packet_rate":0,"peak_output_data_rate":0,"peak_output_packet_rate":0,"reliability":255},"name":"data_rate","tags":{"host":"86cab7c92999","interface_name":"GigabitEthernet0/0/0/2","path":"Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/data-rate","source":"A9Kv-6","subscription":"Sub1"},"timestamp":1596189500}
```

#### generic_counters

```
{"fields":{"applique":0,"availability_flag":0,"broadcast_packets_received":58690043,"broadcast_packets_sent":0,"bytes_received":6330562129,"bytes_sent":182100399403,"carrier_transitions":3,"crc_errors":0,"framing_errors_received":0,"giant_packets_received":0,"input_aborts":0,"input_drops":0,"input_errors":0,"input_ignored_packets":0,"input_overruns":0,"input_queue_drops":0,"last_data_time":1596189650,"last_discontinuity_time":1594109585,"multicast_packets_received":0,"multicast_packets_sent":0,"output_buffer_failures":0,"output_buffers_swapped_out":0,"output_drops":0,"output_errors":0,"output_queue_drops":0,"output_underruns":0,"packets_received":102551903,"packets_sent":121667060,"parity_packets_received":0,"resets":0,"runt_packets_received":0,"seconds_since_last_clear_counters":0,"seconds_since_packet_received":0,"seconds_since_packet_sent":0,"throttled_packets_received":0,"unknown_protocol_packets_received":0},"name":"generic_counters","tags":{"host":"86cab7c92999","interface_name":"MgmtEth0/RP0/CPU0/0","path":"Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters","source":"A9Kv-6","subscription":"Sub1"},"timestamp":1596189650}
```

#### cpu_utilization

```
{"fields":{"process_cpu/process_cpu_fifteen_minute":0,"process_cpu/process_cpu_five_minute":0,"process_cpu/process_cpu_one_minute":0,"process_cpu/process_id":24451,"process_cpu/process_name":"exec","process_cpu/thread_cpu/process_cpu_fifteen_minute":0,"process_cpu/thread_cpu/process_cpu_five_minute":0,"process_cpu/thread_cpu/process_cpu_one_minute":0,"process_cpu/thread_cpu/thread_id":24465,"process_cpu/thread_cpu/thread_name":"evm_signal_thre","total_cpu_fifteen_minute":1,"total_cpu_five_minute":2,"total_cpu_one_minute":2},"name":"cpu_utilization","tags":{"host":"86cab7c92999","node_name":"0/RP0/CPU0","path":"Cisco-IOS-XR-wdsysmon-fd-oper:system-monitoring/cpu-utilization","source":"A9Kv-6","subscription":"Sub1"},"timestamp":1596189655}
```

### InfluxDB CLI

```
Connected to http://localhost:8086 version 1.8.1
InfluxDB shell version: 1.8.1
> select "bytes_sent","bytes_received","interface_name" from "generic_counters" where time > now() - 15s
name: generic_counters
time                     bytes_sent   bytes_received interface_name
----                     ----------   -------------- --------------
2020-07-31T10:26:15.4Z   17958093757  10720465560    GigabitEthernet0/0/0/0
2020-07-31T10:26:15.4Z   0            0              Null0
2020-07-31T10:26:15.4Z   24927755     7158889289     GigabitEthernet0/0/0/1
2020-07-31T10:26:15.4Z   7081436276   7143072418     GigabitEthernet0/0/0/3
2020-07-31T10:26:15.4Z   182484983591 6339236225     MgmtEth0/RP0/CPU0/0
2020-07-31T10:26:15.4Z   0            0              GigabitEthernet0/0/0/4
2020-07-31T10:26:15.4Z   10642648337  10621234638    GigabitEthernet0/0/0/2
2020-07-31T10:26:15.4Z   0            0              GigabitEthernet0/0/0/5
2020-07-31T10:26:15.4Z   0            0              GigabitEthernet0/0/0/6
2020-07-31T10:26:20.409Z 0            0              GigabitEthernet0/0/0/5
2020-07-31T10:26:20.409Z 0            0              GigabitEthernet0/0/0/6
2020-07-31T10:26:20.409Z 17958159180  10720503650    GigabitEthernet0/0/0/0
2020-07-31T10:26:20.409Z 0            0              GigabitEthernet0/0/0/4
2020-07-31T10:26:20.409Z 182486245359 6339267416     MgmtEth0/RP0/CPU0/0
2020-07-31T10:26:20.409Z 24927829     7158916285     GigabitEthernet0/0/0/1
2020-07-31T10:26:20.409Z 0            0              Null0
2020-07-31T10:26:20.409Z 10642686161  10621272462    GigabitEthernet0/0/0/2
2020-07-31T10:26:20.409Z 7081463006   7143099210     GigabitEthernet0/0/0/3
> select "input_data_rate","output_data_rate","interface_name" from "data_rate" where time > now() - 30s
name: data_rate
time                     input_data_rate output_data_rate interface_name
----                     --------------- ---------------- --------------
2020-07-31T14:54:55.772Z 0               0                Null0
2020-07-31T14:54:55.774Z 43              2006             MgmtEth0/RP0/CPU0/0
2020-07-31T14:54:55.776Z 0               0                SINT0/0/0
2020-07-31T14:54:55.777Z 56              94               GigabitEthernet0/0/0/0
2020-07-31T14:54:55.778Z 0               0                GigabitEthernet0/0/0/6
2020-07-31T14:54:55.78Z  0               0                GigabitEthernet0/0/0/5
2020-07-31T14:54:55.782Z 0               0                GigabitEthernet0/0/0/4
2020-07-31T14:54:55.784Z 37              37               GigabitEthernet0/0/0/3
2020-07-31T14:54:55.785Z 55              56               GigabitEthernet0/0/0/2
2020-07-31T14:54:55.786Z 38              0                GigabitEthernet0/0/0/1
2020-07-31T14:55:00.766Z 0               0                Null0
2020-07-31T14:55:00.768Z 0               0                SINT0/0/0
2020-07-31T14:55:00.769Z 56              94               GigabitEthernet0/0/0/0
2020-07-31T14:55:00.771Z 0               0                GigabitEthernet0/0/0/6
2020-07-31T14:55:00.772Z 0               0                GigabitEthernet0/0/0/5
2020-07-31T14:55:00.773Z 0               0                GigabitEthernet0/0/0/4
2020-07-31T14:55:00.774Z 38              37               GigabitEthernet0/0/0/3
2020-07-31T14:55:00.775Z 55              56               GigabitEthernet0/0/0/2
2020-07-31T14:55:00.776Z 38              0                GigabitEthernet0/0/0/1
2020-07-31T14:55:00.777Z 43              2006             MgmtEth0/RP0/CPU0/0
> select "total_cpu_one_minute","node_name" from "cpu_utilization" where time > now() - 45s
name: cpu_utilization
time                     total_cpu_one_minute node_name
----                     -------------------- ---------
2020-07-31T15:08:50.667Z 2                    0/RP0/CPU0
2020-07-31T15:08:50.718Z 28                   0/0/CPU0
2020-07-31T15:08:55.649Z 2                    0/RP0/CPU0
2020-07-31T15:08:55.701Z 28                   0/0/CPU0
2020-07-31T15:09:00.658Z 2                    0/RP0/CPU0
2020-07-31T15:09:00.703Z 28                   0/0/CPU0
2020-07-31T15:09:05.641Z 2                    0/RP0/CPU0
2020-07-31T15:09:05.688Z 28                   0/0/CPU0
2020-07-31T15:09:10.642Z 2                    0/RP0/CPU0
2020-07-31T15:09:10.679Z 28                   0/0/CPU0
```

### Grafana Dashboard

TBD

## About Cisco IOS-XR Model-Driven Telemetry

Please refer this [link](https://xrdocs.io/telemetry/) for more information about Cisco IOS-XR MDT.

## To Do

Add Cisco IOS-XE and Cisco NX-OS related things.
