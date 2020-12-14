---
layout: post
title: What happens if you change sysctl from inside a pod?
date: 2020-12-14 14:00:00
tags: [frontpage]
categories: [kubernetes]
---


You cannot modify any sysctl setting from a pod that doesn't run under privileged
security context. You would see `Read-only file system` error.


```
=== pod spec ===
# cat no-privilege-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-privilege-pod
spec:
  containers:
  - name: busybox
    image: busybox:1.29.3
    args:
    - sleep
    - "1000000"


=== Create the pod ===
# kubectl apply -f no-privilege-pod.yaml
pod/no-privilege-pod created


=== Modify the sysctl inside pod ===
# kubectl exec -it no-privilege-pod sh
/ # sysctl vm.max_map_count
vm.max_map_count = 65530
/ # sysctl vm.max_map_count=65540
sysctl: error setting key 'vm.max_map_count': Read-only file system
/ # sysctl net.ipv4.tcp_keepalive_intvl
net.ipv4.tcp_keepalive_intvl = 75
/ # sysctl net.ipv4.tcp_keepalive_intvl=70
sysctl: error setting key 'net.ipv4.tcp_keepalive_intvl': Read-only file system
/ #
```


When `spec.securityContext.privileged: True` is set, you can tune the kernel parameters.
If the sysctl setting is namespaced like `net.ipv4.tcp_keepalive_intvl`, it would change
within the pod and doesn't affect the setting on the node. If the sysctl setting is a
node-level parameter like `vm.max_map_count`, changing in the pod would change it on the
node as well.

```
=== pod spec ===
# cat privileged-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: busybox
    image: busybox:1.29.3
    args:
    - sleep
    - "1000000"
    securityContext:
       privileged: true


=== Create the pod ===
# kubectl apply -f privileged-pod.yaml
pod/privileged-pod created


=== Modify the sysctl inside pod ===
# kubectl exec -it privileged-pod sh
/ # sysctl net.ipv4.tcp_keepalive_intvl=70
net.ipv4.tcp_keepalive_intvl = 70
/ # sysctl vm.max_map_count=65540
vm.max_map_count = 65540


=== Check on host ===
=== namespaced sysctl not changed on host ===
# sysctl net.ipv4.tcp_keepalive_intvl
net.ipv4.tcp_keepalive_intvl = 75

=== node-level sysctl is changed on host ===
# sysctl vm.max_map_count
vm.max_map_count = 65530
```