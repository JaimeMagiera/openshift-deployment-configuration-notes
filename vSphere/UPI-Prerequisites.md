# Prerequites for vSphere UPI

In this example I describe the setup of a DNS/DHCP server and a Load Balancer on a Raspberry PI microcomputer. The instructions most certainly will also work for other environments.

I use Raspberry Pi OS (debian based).

## IP Addresses of components in this example
* Homelab subnet: 192.168.178.0/24
* DSL modem/gateway: 192.168.178.1
* IP address of Raspberry Pi (DHCP/DNS/Load Balancer): 192.168.178.5
* local domain: homelab.net
* local cluster (name: c1) domain: c1.homelab.net
* DHCP range: 192.168.178.40 ... 192.168.178.199
* Static IPs for k8s bootstrap, masters and workers

## Set static IP address on Raspberry Pi
/etc/dhcpcd.conf
```
interface eth0
static ip_address=192.168.178.5/24
static routers=192.168.178.1
static domain_name_servers=192.168.178.5
```

## DHCP
Ensure that no other DHCP servers are activated in the network of your homelab e.g. in your internet router.

### Install

```
sudo apt-get install isc-dhcp-server
```

### Configure

/etc/dhcp/dhcpd.conf
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

## DNS

### Install

```
sudo apt install bind9 dnsutils
```

### Configure

/etc/bind/named.conf.options
```
include "/etc/bind/rndc.key";

acl internals {
    // lo adapter
    127.0.0.1;

    // CIDR for your homelab network
    192.168.178.0/24;
};

options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
          8.8.8.8;
          8.8.4.4;
        };
        forward only;

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation no;

        listen-on-v6 { none; };
        auth-nxdomain no;
        listen-on port 53 { any; };

        // Allow queries from my Homelab and also from Wireguard Clients.
        allow-query { internals; };
        allow-query-cache { internals; };
        allow-update { internals; };
        recursion yes;
        allow-recursion { internals; };
        allow-transfer { internals; };

        dnssec-enable no;

        check-names master ignore;
        check-names slave ignore;
        check-names response ignore;

};
```

/etc/bind/named.conf.local
```
#include "/etc/bind/rndc.key";

//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

# All devices that don't belong to the OKD cluster will be maintained here.
zone "homelab.net" {
   type master;
   file "/etc/bind/forward.homelab.net";
   allow-update { key rndc-key; };
};

zone "c1.homelab.net" {
   type master;
   file "/etc/bind/forward.c1.homelab.net";
   allow-update { key rndc-key; };
};
```

## Load Balancer

### Install

```
sudo apt-get install haproxy
```


## Proxy (if on a private network)
