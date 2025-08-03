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
Setting up OSPF for UNDERLAY 
```
router ospf UNDERLAY
  router-id 10.2.0.X
!
interface loopback0
 ip router ospf UNDERLAY area 0.0.0.0 
```
Configure Interfaces on SPINES - towards LEAFS (eth1/1-3)
(ip unnumbered loopback0)
```
interface Ethernet1/1-3
  no shutdown
  no switchport
  medium p2p
  mtu 9216
  ip address 10.4.0.X/30
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
```

Configure Interfaces on LEAFs - towards SPINES (eth1/1-2 )
```
interface Ethernet1/1-2
  no shutdown
  no switchport
  medium p2p
  mtu 9216
  ip address 10.4.0.X/30
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
```

Verfication Commands
```
show ip ospf neighbor
!
show ip route ospf-UNDERLAY
!
```
<img width="807" height="730" alt="image" src="https://github.com/user-attachments/assets/de276d2e-3b40-45b3-81e6-ed1e1062d1b7" />

<img width="805" height="726" alt="image" src="https://github.com/user-attachments/assets/8e240959-c75c-4d5b-b04d-d134149fe970" />


Setting up Multicast - PIM 
loopback1 (on spines only)
loopback0 (config on leafs)

SPINE config
```
ip pim rp-address 10.2.0.99 group-list 224.0.0.0/4
ip pim ssm range 232.0.0.0/8
ip pim anycast-rp 10.2.0.99 10.2.0.1
ip pim anycast-rp 10.2.0.99 10.2.0.2
interface Loopback1
 ip address 10.2.0.99/32
 ip router ospf UNDERLAY area 0.0.0.0
 ip pim sparse-mode
```

LEAF config
```
ip pim rp-address 10.0.0.99 group-list 224.0.0.0/4
ip pim ssm range 232.0.0.0/8
```

SPINES
Enable IP PIM Sparse Mode on all interfaces (Lo0,Lo1,intEther1/1-3)
```
ip pim sparse-mode
```

LEAFs
Enable IP PIM Sparse Mode on all interfaces (Lo0,intEther1/1-2)
```
ip pim sparse-mode
```



