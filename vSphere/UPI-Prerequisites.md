# Prerequites for vSphere UPI

In this example I describe the setup of a DNS/DHCP server and a Load Balancer on a Raspberry PI microcomputer. The instructions most certainly will also work for other environments.

I use Raspberry Pi OS (debian based).

## IP Addresses of components in this example
* DSL modem/gateway: 192.168.178.1
* IP address of Raspberry Pi (DHCP/DNS/Load Balancer): 192.168.178.5

## Set static IP address on Raspberry Pi
In /etc/dhcpcd.conf:
```
interface eth0
static ip_address=192.168.178.5/24
static routers=192.168.178.1
static domain_name_servers=192.168.0.2 8.8.8.8
```

## DHCP

### Install

```
sudo apt-get install isc-dhcp-server
```




### Configure

## DNS

### Install

```
sudo apt install bind9
```

### Configure

## Load Balancer

### Install

```
sudo apt-get install haproxy
```


## Proxy (if on a private network)
