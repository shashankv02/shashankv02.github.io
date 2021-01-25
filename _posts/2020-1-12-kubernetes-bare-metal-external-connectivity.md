---
layout: post
title: Exposing services externally on bare metal kubernetes
date: 2020-1-12 12:00:45
tags: [frontpage]
categories: [kubernetes]
---

Exposing services externally on Kubernetes clusters provided by cloud providers
is as simple as setting `spec.type` as `LoadBalancer` in the service
specification and the cloud provider takes care of spinning up a network load
balancer. It is not that simple on bare metal Kubernetes clusters as there is no
support for `LoadBalancer` service out of the box as of today. I had this
problem recently at work and figured out few approaches to deal with this after
some google-fu and researching on how IngressControllers like nginx deal with
it.

**1. NodePort service**

NodePort type service exposes the service on each node’s IP at a static port.
You’ll be able to contact the NodePort Service, from outside the cluster at`<NodeIP>:<NodePort>`. 
You can use any NodeIP in the cluster. NodePort is reserved on all the nodes.

Using a NodePort service as simple of setting `spec.type` as `NodePort` You can
select a custom node port within the node port range with `nodePort` field in
the `ports` object. If `nodePort` field is not specified, kubernetes chooses as
random port within the node port range.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080    # Custom nodePort
      name: http
  selector:
    name: nginx
```

The node port range by default is `30000-32767` and can be configured using
`--service-node-port-range` flag of `kube-api-server`.

If you want to use a low range port like `8080` as node port, you are out of
luck. Though you can change node port range using `--service-node-port-range`
flag, it is not recommended to set it to low range.

**2. HostPort/HostPort with DaemonSet**

You can make a container bind to a port of the host machine using `hostPort`
field in `ports` object of pod specification(or pod template of Deployment,
Statefuleset etc). You will be able to access this container at
`<HostIP>:<HostPort>`. The HostIP is the IP of the node where this pod is
scheduled. Scheduler removes the nodes on which `hostPort` is not free when
trying to select a node for scheduling the pod, so there is a chance that your
pod might not be scheduled at all if the hostPort is not free on any node.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
      hostPort: 80      # hostPort
  restartPolicy: Always
```

This indeterminism of the hostIP till the pod is scheduled is a drawback with
this approach. You can workaround this in by forcing the pod to be scheduled on
a particular node using `nodeAffinity`. But that only creates more issues when
there is node outage. Either the pod might not get scheduled at all depending on
`nodeAffinity` configuration and if it gets scheduled you would need to find
the IP of the new node.

Another way to workaround this for some workloads might be to use a
[`DaemonSet`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).
`DaemonSet` ensures that all nodes in the cluster run a replica of the pod. So
you can use any <HostIP>:<HostPort> to reach a replica of your pod. Note that
there is no loadbalancing across the replicas of the pods in thie approach. If
external clients connect to same node every time, the pod on that node would
receive all the traffic. This might be okay for workloads like lightweight
proxies etc. You can also configure load balancing at external DNS so that
clients receive differnt node IP each time.

**3. Metallb**

[MetalLB](https://metallb.universe.tf/) is a load-balancer implementation for
bare metal Kubernetes clusters. Metallb deploys a couple of services on the
cluster to do it's magic. You can give metallb a pool of virtual IP addresses
that are routable to the cluster to handout and it takes care of assigning these
IP addresses to the `LoadBalancer` type services.

You can read more about installing and configuring metallb in the [this post]({% post_url 2020-1-12-metallb-layer2-mode %}).

**4. Ingress Controllers**

`Ingress` resource acts like a HTTP(S) rerverse proxy. It can route requests to
one of your backend services running in the cluster based on the request path.
Instead of deploying a reverse proxy like nginx yourself and writing the rules,
you can use `Ingress`.

`Ingress` will work only if there is an `IngressController` installed in the
cluster. If you dealing with HTTP traffic only, then you can install an
`IngressController`. `IngressController` itself cannot expose your service
externally. As part of the installation, most Ingress Controllers allow you to
choose one of the methods described above to receive external traffic into the
cluster.

_References_

[https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)

[https://stackoverflow.com/questions/59628796/how-to-access-kubernetes-services-externally-on-bare-metal-cluster](https://stackoverflow.com/questions/59628796/how-to-access-kubernetes-services-externally-on-bare-metal-cluster)

[https://metallb.universe.tf/](https://metallb.universe.tf/)

[https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)

[https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
