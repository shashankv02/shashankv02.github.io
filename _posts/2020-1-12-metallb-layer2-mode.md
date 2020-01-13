---
layout: post
title: Running metallb in Layer2 mode
date: 2020-1-12 13:00:45
tags: [frontpage]
categories: [kubernetes]
---

[MetalLB](https://metallb.universe.tf/) is a load-balancer implementation for
bare metal Kubernetes clusters. I've used it succesfully to add support for
`LoadBalancer` type service on bare-metal kubernetes clusters to expose my
services outside the cluster.

**Installation**

First check if metallb is compatible with your CNI plugin at
[https://metallb.universe.tf/installation/network-addons/](https://metallb.universe.tf/installation/network-addons/).

Installing `metallb` is as simple as running the following command.

`kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.3/manifests/metallb.yaml`.

This deploys two components - controller and speaker. controller is deployed as
a Kubernetes Deployment and speaker is deployed as a Kubernetes Daemonset.
Metallb deals with two primary tasks - Address allocation and external
anouncement.

Once metallb is installed, you configure it by creating a `ConfigMap` object
under the same namespace it is installed in(metallb-system). Metallb works in
two modes - Layer2 and BGP. I've used it in Layer2 as it is simplest to configure.

Following is example of configuration for Layer2 mode configuration:

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      # Change this address pool to your IP range
      - 192.168.1.240-192.168.1.250
```

**How does it work?**

The `controller` component of metallb watches for creation or updation of
kubernetes Service objects of type `LoadBalancer` and assigns an IP from the
address pool given to it from the configuration.

The job of `speaker` component is to announce this address. `speaker` is deployed as a
Daemonset and so runs on every node. When `controller` assigns an IP to a
`speaker`, it announces this event by sending out gratuitous ARP messages and by
replying to ARP requests.

If a node that owns the IP goes down, another node takes over the IP and starts
responding to the ARP requests.

One disadvantage with Layer 2 mode is that a single node attracts all the
service IP's traffic. From there, kube-proxy spreads the traffic across the
service's pods. So there is failover between nodes but there is no node level
loadbalancing. So the service's ingress bandwidth is limited to the bandwidth of
single node.

**What virtual IPs should I use?**

In Layer 2 mode, the virtual IPs that you give to metallb should be routable to the cluster
network. This means all the nodes IPs and the extra virtual IPs should be in same
subnet.

**How many virtual IPs do I need?**

If you are running inside a private network, it should be easy to get as
many IP addresses as you want. At minimum you would need as many IPs as the
number of `LoadBalancer` services you have so that each service get's it's own IP.

**Sharing IPs among different services**

If for some reason, you do not have the luxury of getting the reuired number of
IPs or if you want to expose all the services on same IP, metallb supports IP
sharing mode where multiple services can use same IP given that they use
different service ports. To enable IP sharing, you need add an annotation with
key `metallb.universe.tf/allow-shared-ip` and use same value for this annotation
in all the services you want to collocate on same IP. To guarantee that IP is
shared among the services, you also need to set the `spec.loadBalancerIP` field
to required IP. If only annotation is set and `spec.loadBalancerIP` is not set,
metallb tried to collacate the services on same IP but it is not guranteed.

```
apiVersion: v1
kind: Service
metadata:
  name: gateway
  labels:
    app: gateway
  annotations:
    # Use `my-shared-ip` value in other services which
    # should be collocated with this service
    metallb.universe.tf/allow-shared-ip: my-shared-ip
spec:
  type: LoadBalancer
  # Set loadBalancerIP to guarantee services share the IP
  loadBalancerIP: 192.168.1.240
  selector:
    app: gateway
  - name: http
    port: 80
    targetPort: 80
```
