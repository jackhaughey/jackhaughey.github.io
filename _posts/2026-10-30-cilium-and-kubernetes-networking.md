# Understanding Cilium, eBPF, Envoy and Kubernetes Networking

While building my three-node Kubernetes cluster using Rocky Linux, kubeadm and Cilium, I reached the point where the cluster was healthy and all nodes were reporting Ready.

At that stage I started investigating how networking actually worked beneath the surface.

Coming from a networking and telecoms background, I initially assumed there would be a single network carrying traffic throughout the cluster. What I discovered was that Kubernetes and Cilium operate across multiple networking layers simultaneously.

## Three Networks Inside the Cluster

My infrastructure nodes live on the physical LAN:

```
10.2.0.0/24

k8s-controller-01  10.2.0.220
k8s-worker-01      10.2.0.221
k8s-worker-02      10.2.0.222
```

These addresses are used for:

- SSH
- DNS
- FreeIPA
- kubelet communication
- API server communication
- Node-to-node traffic

However, Kubernetes introduces additional networks.

## Pod Network

The CoreDNS pods in my cluster received addresses such as:

```
10.0.0.232
10.0.0.241
```

These addresses do not exist on my physical LAN.

They belong to the Kubernetes pod network managed by Cilium.

## Service Network

Running:

```
kubectl get services -A
```

Revealed more networks;

```
10.96.0.1
10.96.0.10
10.111.69.99
```

These are Service VIPs.

For example:

```
kube-dns
    ↓
10.96.0.10
```

is a virtual IP that forwards traffic to the CoreDNS pods.

Applications don't connect directly to:

```
10.0.0.232
```

Instead they connect to:

```
10.96.0.10
```

and Kubernetes performs the load balancing.

## Where Cilium Fits

One thing I noticed during installation was that Cilium deployed both ```cilium``` and ```cilium-pods``` pods.

Initially I assumed Envoy was responsible for all networking traffic.

It turns out that isn't how Cilium works.

## eBPF Handles Most Traffic

Cilium's primary datapath is based on eBPF.

Most cluster traffic follows a path similar to:

```
Pod
 ↓
eBPF
 ↓
Pod
```

or;

```
Pod
 ↓
Service
 ↓
Pod
```

without involving Envoy at all.

Using eBPF allows Cilium to perform:

- Routing
- Service load balancing
- NAT
- Network policy enforcement
- Packet forwarding

directly within the Linux kernel.

This is one of the reasons Cilium can replace large amounts of iptables processing.

## Why Cilium Uses Envoy

While eBPF is excellent at Layers 3 and 4, it doesn't naturally understand application protocols.

For example, eBPF can identify:

```
TCP port 443
```

but cannot easily understand:

```
GET /health
POST /admin
HTTP Headers
gRPC Methods
```

For those use cases, Cilium integrates Envoy.

The traffic flow becomes:

```
Application
 ↓
Envoy
 ↓
Application
```

allowing Layer 7 inspection and policy enforcement.

This enables features such as:

- HTTP-aware network policies
- Gateway API
- Ingress
- Advanced traffic routing
- Observability

## One Envoy Per Node, Not Per Application

Another interesting discovery was that Envoy is not deployed beside every workload.

My cluster currently contains:

```
3 Kubernetes Nodes
3 Cilium Envoy Pods
```

because Cilium deploys Envoy as a DaemonSet.

The architecture therefore looks more like:

```
Node
├── Cilium
├── Envoy
└── Workloads
```

rather than:

```
Workload
└── Envoy
```

for every single application.

This keeps resource consumption significantly lower than traditional sidecar-heavy service mesh designs.

## The Key Takeaway

The most useful mental model I found was:

```
Cilium
│
├── eBPF
│   └── Layer 3 / Layer 4
│
└── Envoy
    └── Layer 7
```

or;

```
eBPF = Fast packet forwarding

Envoy = Application-aware proxy
```

Together they provide a modern networking platform where most traffic is handled efficiently inside the Linux kernel while still allowing deep inspection when application awareness is required.

For someone coming from a networking background, understanding how the:

```
Node Network
Pod Network
Service Network
```

interact through Cilium has been one of the most valuable lessons from building this Kubernetes cluster.
