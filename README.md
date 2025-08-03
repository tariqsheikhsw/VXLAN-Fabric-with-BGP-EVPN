# VXLAN-Fabric-with-BGP-EVPN
VXLAN-Fabric-with-BGP-EVPN

# Use-Case:1 SPINE LEAF ARCHITECTURE 

# L2 VNIs
vlan10 - l2vni 100010  
vlan20 - l2vni 100020   

# L3 VNIs
vlan999 - OVERLAY-TENANT1 - l3vni 100999  
vlan998 - OVERLAY-TENANT2 - l3vni 100998  

# Topology (CML)

<img width="1058" height="452" alt="cmle" src="https://github.com/user-attachments/assets/b8c16e11-f85b-4346-9f1c-fc6796e4d3b1" />

# Configuration Breakdown 

To avoid ARP suppression configuration related issues

```
hardware access-list tcam region vpc-convergence 0
hardware access-list tcam region arp-ether 256 
```

NXOS Features (Spine and Leaf) - OSPF is for UNDERLAY and BGP is for OVERLAY
```
feature ospf
feature bgp
feature pim
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay
```

Loopback0 - VTEP / BGP peerings
Spine1 - 10.2.0.1/32
Spine2 - 10.2.0.2/32
Leaf1 - 10.2.0.11/32
Leaf2 - 10.2.0.22/32
Leaf3 - 10.2.0.33/32

```
interface loopback0
  ip address 10.2.0.X/32
```





