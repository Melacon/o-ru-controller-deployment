# O-RU Controller Deployment

This project focus on a docker-compose deployment solution for O-RU Controller Components.

## Introduction

An O-RU Controller is defined by O-RAN Alliance WG4. It consumes the OpenFronthaul 
Management-Plane API and implements a NETCONF Client for configuration.

The setup contains an OpenDaylight based NETCONF client and an ONAP VES Collector.

This deployment focus on the very basic functions and uses port mappings instead for FQDNs. 

## O-RU Components

This docker-compose file starts a pre-configured, self-contained SDN-R solution
with the following components.


  * **Controller** single node instance

    ... representing the NETCONF consumer on the O-RU Controller based on
    OpenDaylight.
    The Controller comes with is own web-portal the external port is 8453 and
    with a flow-editor at port 1880.

  * **VES collector**

    ... representing the VES (REST) provider at O-RU Controller for all kind of events. In this configuration the external https port is 8443.

  * **Messages**
    ... representing a message-router used for O-RU Controller internal communication.

## Prerequisites

### Hardware or virtual machine

The solution required at least 4 cores, 16 Gbit RAM and 50 Gbit of storage. 


### Software

The following versions were used during development and test. 

```
$ cat /etc/os-release | grep PRETTY_NAME
PRETTY_NAME="Ubuntu 22.04.2 LTS"

$ docker --version
Docker version 23.0.4, build f480fb1

$ docker compose version
Docker Compose version v2.16.0

$ git --version
git version 2.34.1

$ python3 --version
Python 3.10.6

$ docker ps -a --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
NAMES                    IMAGE                                                                                        STATUS
messages                 nexus3.onap.org:10001/onap/dmaap/dmaap-mr:1.1.18                                             Up 15 minutes
flows                    nodered/node-red:latest                                                                      Up 13 minutes (healthy)
odlux                    nexus3.onap.org:10001/onap/sdnc-web-image:2.4.2                                              Up 13 minutes
kafka                    nexus3.onap.org:10001/onap/dmaap/kafka111:1.0.4                                              Up 15 minutes
zookeeper                nexus3.onap.org:10001/onap/dmaap/zookeeper:6.0.3                                             Up 15 minutes
persistence              docker.elastic.co/elasticsearch/elasticsearch-oss:7.9.3                                      Up 15 minutes
ves-collector            nexus3.onap.org:10001/onap/org.onap.dcaegen2.collectors.ves.vescollector:1.10.1-configured   Up 15 minutes (healthy)
controller               nexus3.onap.org:10001/onap/sdnc-image:2.4.2                                                  Up 15 minutes (healthy)
ntsim-ng-o-ru-fh-11221   nexus3.o-ran-sc.org:10004/o-ran-sc/nts-ng-o-ran-ru-fh:1.5.2                                  Up 5 hours
ntsim-ng-o-ru-fh-11223   nexus3.o-ran-sc.org:10004/o-ran-sc/nts-ng-o-ran-ru-fh:1.5.2                                  Up 5 hours
ntsim-ng-o-ru-fh-11222   nexus3.o-ran-sc.org:10004/o-ran-sc/nts-ng-o-ran-ru-fh:1.5.2                                  Up 5 hours

```
For IPV6 support see the following instructions and update your/etc/docker/daemon.json accordingly
https://docs.docker.com/config/daemon/ipv6/

It is beneficial (but not mandatory) adding the following line add the
end of your ~/.bashrc file. I will suppress warnings when python script
do not verify self signed certificates for HTTPS communication.

```
export PYTHONWARNINGS="ignore:Unverified HTTPS request"
```


## Usage

### Bring Up Solution

Please get familiar with the environment variables to bring up the solution
on your system.

```
nano o-ru-controller/.env
nano network/.env
```
At minimum configure the IPv4 Address:
* SDNC_OAM_IPv4=aaa.bbb.ccc.ddd
* VES_COLLECTOR_OAM_IPv4=aaa.bbb.ccc.ddd
to the interface your system can be reached from external machines. 

The following commands should be invoked. More detailed can be found in the
next chapters.

```
docker compose -f o-ru-controller/docker-compose.yml up -d
docker compose -f network/docker-compose.yml up -d
```

### Log files and karaf console

#### ODL karaf.logs

```
docker exec -it controller tail -f /opt/opendaylight/data/log/karaf.log
```

#### ves-collector logs

```
docker logs -f ves-collector
```


#### Login into the O-RU Controller

    https://<your-host-ipv4>:8453

    User: admin // see .env file
    Password: Kp8bJ4SXszM0WXlhak3eHlcse2gAw84vaoGGmJvUy2U

#### Login into the Flow Editor

    https://<your-host-ipv4>:1880

    User: admin // see .env file
    Password: Kp8bJ4SXszM0WXlhak3eHlcse2gAw84vaoGGmJvUy2U

### Terminate solution

To stop all container please respect the following order

```
docker-compose -f network/docker-compose.yml down
docker-compose -f o-ru-controller/docker-compose.yml down
```

### Cleanup

!!! be careful if other stopped containers are on the same system
```
docker system prune -a -f
```

### Troubleshooting

In most cases the .env setting do not fit to the environment and need to be
adjusted.

Please make sure that the network settings to not overlap with other networks.

The commands ...
```
docker ps -a
docker compose ps
