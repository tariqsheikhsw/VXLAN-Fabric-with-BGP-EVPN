# VXLAN-Fabric-with-BGP-EVPN
VXLAN-Fabric-with-BGP-EVPN

# Getting Started
To build a VXLAN fabric, follow these steps:

# Pre-Requisites:
- Run an IGP (Interior Gateway Protocol) of your choice between Spine and Leaf switches to ensure all loopbacks are available.
- Enable Multicast and Configure Anycast RP on the Spine switch.
- Configure MSDP (Multicast Source Discovery Protocol) between Spine switches to share active source information across different multicast domains.

# VXLAN Fabric Build Steps:

- Step 1:
Define a Tenant VRF (Virtual Routing and Forwarding) on each Leaf switch. Each leaf must have a unique rd (route distinguisher).

- Step 2:
Configure BGP (Border Gateway Protocol) neighborship between Leaf and Spine switches only for the l2vpn address-family. Here, the spine acts as a Route-reflector. Also, on leafs, advertise the L2VPN routes out to the VRF address-family.

- Step 3:
Configure the replication type on all Leaf switches. For this demo, we will use Static replication.

- Step 4:
Configure L2VNI (Layer 2 VxLAN Network Identifier) and VXLAN to VLAN mapping on Leafs, to extend this Layer L2VNI across all the leaf switches.

- Step 5:
Configure Distributed Anycast Gateway for the Access VLANs on Leafs. Ensure the same IP address and mac-address are present on all DAGs (Distributed Anycast Gateways).

- Step 6:
Define L3VNI and corresponding VLANs on all Leafs. A L3VNI is used to route traffic between different VNIs/Subnets.

- Step 7:
Configure an SVI (Switched Virtual Interface) for the L3VNI and assign it to the Tenant VRF.

- Step 8:
Create a Network Virtualization Endpoint (NVE) interface. Map L2VNIs and L3VNI to the NVE interface. This is where the encapsulation and decapsulation happens.


### VXLAN EVPN CONFIG CONSTRUCT w/ SAMPLE CONFIG

# Replication Method
```
l2vpn evpn
 router-id Loopback0
 replication-type static
 ! defining static multicast replication here, used for BUM traffic
end
```

# L2VNI
```
l2vpn evpn instance <ID> vlan-based
 encapsulation vxlan
 !Defining VXLAN encapsulation here with a specific instance.
 exit
 vlan configuration <DAG-VLAN-ID>
  member evpn-instance <ID> vni <L2VNI-ID>
 !VXLAN VNI to VLAN mapping with above instance ID
 exit
```

# L3VNI
```
vlan configuration <L3VNI-CORE-VLAN-ID>
 member vni <L3VNI-ID>
!
interface Vlan <L3VNI-CORE-VLAN-ID>
 vrf forwarding <VRF-NAME>
 ip unnumbered Loopback0
exit
```

# NVE
```
interface nve1
 source-interface lo0
 host-reachability protocol bgp
 member vni <L2VNI-ID> mcast-group <Group-ID-1>
 member vni <L2VNI-ID> mcast-group <Group-ID-2>
 member vni <L3VNI-ID> vrf <VRF-NAME>
end
```

# Dynamic Anycast Gateways (DAG SVI)
```
interface Vlan<VLAN-ID>
 !Distributed Anycast Gateway (DAG) SVI on Edge
 mac-address <H.H.H>
 vrf forwarding <VRF-NAME>
 ip address <NET><Mask>
 no autostate
```

