# Prerequites for vSphere UPI

## Security Policies in VM Network

Ensure that promiscuous mode is turned off on your VM Network in vSphere. If it is turned on, you will have troubles with OVNKubernetes. 

> Because of a bug in the NetworkManager v1.26 used in OKD 4.6 / OKD 4.7 it may be that after a reset of your VMs they will get wrong IP addresses from the DHCP server. That can break your cluster!

```
Allow promiscuous mode	No
Allow forged transmits	No
Allow MAC changes	    No
```

## DNS

## Load Balancer

## Proxy (if on a private network)
