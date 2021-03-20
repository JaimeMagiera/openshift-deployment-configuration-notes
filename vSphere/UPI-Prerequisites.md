# Prerequites for vSphere UPI

In this example I describe the setup of a DNS/DHCP server and a Load Balancer on a Raspberry PI microcomputer. The instructions most certainly will also work for other environments.

I use Raspberry Pi OS (debian based).

## IP Addresses of components in this example
* Homelab subnet: 192.168.178.0/24
* DSL modem/gateway: 192.168.178.1
* IP address of Raspberry Pi (DHCP/DNS/Load Balancer): 192.168.178.5
* local domain: homelab.net
* DHCP range: 192.168.178.40 ... 192.168.178.199
* Static IPs for k8s bootstrap, masters and workers

## Set static IP address on Raspberry Pi
In /etc/dhcpcd.conf:
```
interface eth0
static ip_address=192.168.178.5/24
static routers=192.168.178.1
static domain_name_servers=192.168.0.5 8.8.8.8
```

## DHCP
Ensure that no other DHCP servers are activated in the network of your homelab e.g. in your internet router.

### Install

```
sudo apt-get install isc-dhcp-server
```

/etc/dhcp/dhcpd.conf:
```
# dhcpd.conf
#

####################################################################################
# Basic configuration                                                              #
####################################################################################
# option definitions common to all supported networks...
default-lease-time 60;
max-lease-time     60;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# Parts of this section will be put in the /etc/resolv.conf of your hosts later
option domain-name "homelab.net";
option routers 192.168.178.1;
option subnet-mask 255.255.255.0;
option domain-name-servers 192.168.178.5;

subnet 192.168.178.0 netmask 255.255.255.0 {
  range 192.168.178.40 192.168.178.199;
}
####################################################################################


####################################################################################
# Static IP addresses                                                              #
# (Replace the MAC addresses here with the ones you set in vsphere for your vms)   #
####################################################################################
group {
  host bootstrap {
      hardware ethernet 00:1c:00:00:00:00;
      fixed-address 192.168.178.200;
  }

  host master0 {
      hardware ethernet 00:1c:00:00:00:10;
      fixed-address 192.168.178.210;
  }

  host master1 {
      hardware ethernet 00:1c:00:00:00:11;
      fixed-address 192.168.178.211;
  }

  host master2 {
      hardware ethernet 00:1c:00:00:00:12;
      fixed-address 192.168.178.212;
  }

  host worker0 {
      hardware ethernet 00:1c:00:00:00:20;
      fixed-address 192.168.178.220;
  }

  host worker1 {
      hardware ethernet 00:1c:00:00:00:21;
      fixed-address 192.168.178.221;
  }
  
  host worker2 {
      hardware ethernet 00:1c:00:00:00:22;
      fixed-address 192.168.178.222;
  }  
}
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
