# VXLAN-Fabric-with-BGP-EVPN
VXLAN-Fabric-with-BGP-EVPN

# Use-Case:1 SPINE LEAF ARCHITECTURE 

# L2 VNIs
vlan10 - l2vni 10010  
vlan20 - l2vni 10020   

# L3 VNIs
vlan999 - OVERLAY-TENANT1 - l3vni 100999  
vlan998 - OVERLAY-TENANT2 - l3vni 100998  

# Topology (CML)

<img width="1024" height="429" alt="473774390-ccb15d23-6a25-4e5a-8240-a61cea213c3c" src="https://github.com/user-attachments/assets/0f5126be-953c-4f9b-b7bb-261aecb2c3ea" />

<img width="1480" height="852" alt="image" src="https://github.com/user-attachments/assets/bc1e6044-18d1-492f-8aa4-75fa8f4a1bad" />


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

Verification Commands
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
ip pim rp-address 10.2.0.99 group-list 224.0.0.0/4
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

Verification Commands
```
show ip pim rp
```

<img width="625" height="482" alt="image" src="https://github.com/user-attachments/assets/69267296-aef8-452a-b9e5-a9ad7f89b4fe" />

<img width="586" height="482" alt="image" src="https://github.com/user-attachments/assets/caf4607a-a642-4c2e-abc6-2d9668049da9" />


Configure NVE (Network Virtual Endpoint) Interface - VTEP 
only required on leafs
```
interface nve1 
 no shutdown
 host-reachability protocol bgp
 source-interface loopback0
```

Verification Commands
```
show interface nve1 
```

<img width="650" height="212" alt="image" src="https://github.com/user-attachments/assets/bfce0caf-53da-4cca-bf16-f2f57fcb176b" />


Enable NV Overlay (all devices)
```
nv overlay evpn 
```

Configure BGP on SPINES
```
router bgp 64500
address-family ipv4 unicast
address-family l2vpn evpn
    retain route-target all
template peer LEAF
    remote-as 64500
    update-source loopback0
    address-family ipv4 unicast
      send-community extended
    route-reflector-client
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
neighbor 10.2.0.11
    inherit peer LEAF
neighbor 10.2.0.22
    inherit peer LEAF
neighbor 10.2.0.33
    inherit peer LEAF
```

Verification Commands
```
show bgp ipv4 unicast summary
```

<img width="945" height="243" alt="image" src="https://github.com/user-attachments/assets/6be8ba20-81e3-4b50-bd25-c2fe96ecf782" />


Configure BGP on LEAFs
```
router bgp 64500
log-neighbor-changes
address-family ipv4 unicast
address-family l2vpn evpn
template peer SPINE
    remote-as 64500
    update-source loopback0
    address-family ipv4 unicast
    send-community extended
    soft-reconfiguration inbound
    address-family l2vpn evpn
    send-community
    send-community extended
neighbor 10.2.0.1
    inherit peer SPINE
neighbor 10.2.0.2
    inherit peer SPINE
```

Verification Commands
```
show bgp ipv4 unicast summary
!
show bgp l2vpn evpn summary
!
```

<img width="1020" height="587" alt="image" src="https://github.com/user-attachments/assets/2fbdf47e-d308-461f-bef7-551d8054f573" />

Configure L2VNIs (on Leafs) - vlan 10 and vlan 20


```
vlan 10
 vn-segment 10010
vlan 20
 vn-segment 10020
```

```
interface nve1
  member vni 10010
    suppress-arp
    mcast-group 224.1.1.192
  member vni 10020
    suppress-arp
    mcast-group 224.1.1.192
```


```
evpn
vni 100010 l2
    rd auto
    route-target import auto
    route-target export auto
vni 100020 l2
    rd auto
    route-target import auto
    route-target export auto
```
//route-target both auto  

ARP suppression verfication
```
show ip arp suppression-cache vlan 10
!
show ip arp suppression-cache vlan 20
!
```

Configure Access VLAN ports on Leafs (connected to End client)
```
interface eth1/1
 switchport
 switchport access vlan 10
 spanning-tree port type edge
 no shut
```


Enduser/Client IP address Verification (10.10.1.X subnet , 10.20.1.X subnet)
```
show bgp l2vpn evpn x.x.x.x
```

Configure AnyCast Gateways (same address on all LEAFs) - on LEAF Switches ONLY 
```
fabric forwarding anycast-gateway-mac aaaa.bbbb.cccc
```

Create SVIs on VLAN10 and VLAN20 
```
interface Vlan10
no shutdown
ip address 10.10.1.254/24
fabric forwarding mode anycast-gateway
!
interface Vlan20
no shutdown
ip address 10.20.1.254/24
fabric forwarding mode anycast-gateway
!
```

Configure L3 VNI 
```
vlan 998
  vn-segment 100998
vlan 999
  vn-segment 100999
```

Configure VRF Context (on LEAFs) 
```
vrf context OVERLAY-TENANT1
  vni 100999
  rd auto
  address-family ipv4 unicast
      route-target both auto
      route-target both auto evpn
```

```
vrf context OVERLAY-TENANT2
  vni 100998
  rd auto
  address-family ipv4 unicast
      route-target both auto
      route-target both auto evpn
```

Associate with NVE1 ( on LEAFs )
```
interface nve1
  member vni 100998 associate-vrf
  member vni 100999 associate-vrf
```

Might need to remove IP and put back in (to configure SVI as part of VRF)
```
interface Vlan10
no shutdown
vrf member OVERLAY-TENANT1
ip address 10.10.1.254/24
fabric forwarding mode anycast-gateway

interface Vlan20
no shutdown
vrf member OVERLAY-TENANT2
ip address 10.20.1.254/24
fabric forwarding mode anycast-gateway
```

Configure SVI as part of VRF
```
interface Vlan998
  no shutdown
  vrf member OVERLAY-TENANT2
  ip forward

interface Vlan999
  no shutdown
  vrf member OVERLAY-TENANT1
  ip forward
```

Because of L3 VNI, we are able to ping from one subnet - vlan 10 (10.10.1.X) to another subnet - vlan20 (10.10.2.X) - from one LEAF to another LEAF



VERIFICATION
```
show bgp vrf OVERLAY-TENANT1 ipv4 unicast
!
show bgp vrf OVERLAY-TENANT1 l2vpn evpn
!
```

Route Leaking 
```
router bgp 64500
  vrf OVERLAY-TENANT1
  address-family ipv4 unicast
    network 10.10.1.0/24
    aggregate-address 10.20.0.0/16 summary-only
  vrf OVERLAY-TENANT2
  address-family ipv4 unicast
    network 10.20.1.0/24
    aggregate-address 10.10.0.0/16 summary-only
```

Configure Route Leaking within VRF 
```
vrf context OVERLAY-TENANT1
  address-family ipv4 unicast
    route-target import 64500:100998
    route-target import 64500:100998 evpn
```

```
vrf context OVERLAY-TENANT2
  address-family ipv4 unicast
    route-target import 64500:100999
    route-target import 64500:100999 evpn
```

VERIFICATION 
```
show bgp vrf OVERLAY-TENANT1 ipv4 unicast
!
show bgp vrf OVERLAY-TENANT1 ipv4 unicast 10.10.1.1/32
!
show bgp vrf OVERLAY-TENANT2 ipv4 unicast
!
show bgp vrf OVERLAY-TENANT1 ipv4 unicast 10.20.1.1/32
!
```


Configure Route Leaking with external VRF (CSR1000V)


```
vlan 555
  vn-segment 100555
```

```
vrf context OVERLAY-TENANT1
  address-family ipv4 unicast
    route-target import 64500:100555
```

```
vrf context OVERLAY-TENANT2
  address-family ipv4 unicast
    route-target import 64500:100555
```

VERIFICATION
```
show ip route vrf external
```


```
vrf context external
  vni 100555
  rd auto
  address-family ipv4 unicast
      route-target both auto
      route-target both auto evpn
      route-target import 64500:100998
      route-target import 64500:100998 evpn
      route-target import 64500:100999
      route-target import 64500:100999 evpn
```

```
interface nve1
  member vni 100555 associate-vrf
```

Create L3 VNI 
```
interface Vlan555
  no shutdown
  vrf member external
  ip forward
```


VERIFICATION
```
show ip route vrf external
```

Configure Interface Connected to CSR1000V 
```
interface Ethernet1/5
  no switchport
  vrf member external
  ip address 10.1.50.1/30
  no shutdown
```

Configure BGP Peering
```
router bgp 64500
  vrf external
    address-family ipv4 unicast
    aggregate-address 10.10.1.0/24 summary-only
    aggregate-address 10.20.1.0/24 summary-only
    neighbor 10.1.50.2
      remote-as 64520
      address-family ipv4 unicast
```

```
vrf context OVERLAY-TENANT1
  address-family ipv4 unicast
    import vrf advertise-vpn
!
```
vrf context OVERLAY-TENANT2
  address-family ipv4 unicast
    import vrf advertise-vpn
!
```

VERIFICATION 
```
show bgp vrf OVERLAY-TENANT1 ipv4 unicast
!
show bgp vrf OVERLAY-TENANT2 ipv4 unicast
!
```

Configure Route Leaking using Route-Maps
```
ROUTE LEAKING / ROUTE MAPS
```

//Some additional configuration required for BGP etc.
Configure Route Leaking using Route-Maps (On CSR1000V)
```
ip prefix-list ONLY-16 permit 0.0.0.0/0 ge 16 le 16
!
route-map ONLY-16 permit 10
 match ip address prefix-list ONL-16
!
router bgp 64520
 neighbor 10.1.50.1 route-map ONLY-16 in
!
```

