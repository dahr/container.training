# Kubernetes network model

- TL,DR:

  *Our cluster (nodes and pods) is one big flat IP network.*

--

- In detail:

 - all nodes must be able to reach each other, without NAT

 - all pods must be able to reach each other, without NAT

 - pods and nodes must be able to reach each other, without NAT

 - each pod is aware of its IP address (no NAT)

 - pod IP addresses are assigned by the network implementation

- Kubernetes doesn't mandate any particular implementation

---

## Kubernetes network model: the good

- Everything can reach everything

- No address translation

- No port translation

- No new protocol

- The network implementation can decide how to allocate addresses

- IP addresses don't have to be "portable" from a node to another

  (We can use e.g. a subnet per node and use a simple routed topology)

- The specification is simple enough to allow many various implementations

---

## Kubernetes network model: the less good

- Everything can reach everything

  - if you want security, you need to add network policies

  - the network implementation that you use needs to support them

- There are literally dozens of implementations out there

  (15 are listed in the Kubernetes documentation)

- Pods have level 3 (IP) connectivity, but *services* are level 4 (TCP or UDP)

  (Services map to a single UDP or TCP port; no port ranges or arbitrary IP packets)

- `kube-proxy` is on the data path when connecting to a pod or container,
  <br/>and it's not particularly fast (relies on userland proxying or iptables)

---

## Kubernetes network model: in practice

- The nodes that we will be using are set up with [PKS](https://pivotal.io/platform/pivotal-container-service) using [NSX-T](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/index.html)

- NSX-T uses the Container Network Interface (CNI) model to provide:

    -   Network Policies
    -   Pod-Level Networking
    -   Load Balancing & Ingress Services
    -   Tenant Isolation
    -   Micro-Segmentation


---

class: title

## PKS & NSX-T

![pksnsx](images/PKS_NSXT.png)

---

class: extra-details

## The Container Network Interface (CNI)

- Most Kubernetes clusters use CNI "plugins" to implement networking

- When a pod is created, Kubernetes delegates the network setup to these plugins

  (it can be a single plugin, or a combination of plugins, each doing one task)

- Typically, CNI plugins will:

  - allocate an IP address (by calling an IPAM plugin)

  - add a network interface into the pod's network namespace

  - configure the interface as well as required routes etc.

---

class: extra-details

## Multiple moving parts

- The "pod-to-pod network" or "pod network":

  - provides communication between pods and nodes

  - is generally implemented with CNI plugins

- The "pod-to-service network":

  - provides internal communication and load balancing

  - is generally implemented with kube-proxy (or e.g. kube-router)

- Network policies:

  - provide firewalling and isolation

  - can be bundled with the "pod network" or provided by another component

---

class: extra-details

## Even more moving parts

- Inbound traffic can be handled by multiple components:

  - something like kube-proxy or kube-router (for NodePort services)

  - load balancers (ideally, connected to the pod network)

- It is possible to use multiple pod networks in parallel

  (with "meta-plugins" like NSX-T)

- Some solutions can fill multiple roles

  (e.g. kube-router can be set up to provide the pod network and/or network policies and/or replace kube-proxy)
