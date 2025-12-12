# VLAN and Inter-VLAN Routing Configuration  

![Topology](Topology.png)

## Objective
Configure a multi-switch campus network with:
- VLAN segmentation (Eng VLAN 10, Sales VLAN 20)
- VTP domain with mixed modes (Server, Transparent, Client)
- Secure trunking (non-default native VLAN)
- Access port assignment
- Inter-VLAN routing using **Router-on-a-Stick** on R1

## Topology Overview
- SW1 → VTP Server  
- SW2 → VTP Transparent + connection to router  
- SW3 → VTP Client  
- VLAN 10 (Eng) – 10.10.10.0/24 – GW 10.10.10.1  
- VLAN 20 (Sales) – 10.10.20.0/24 – GW 10.10.20.1  
- Native VLAN changed to 199 on all trunks (security best practice)

## Final Configuration Summary

### SW1 (VTP Server)
```
vtp domain Flackbox
vtp mode server

interface gig0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 199

interface range f0/1 - 2
 switchport mode access
 switchport access vlan 10
interface f0/3
 switchport mode access
 switchport access vlan 20

vlan 10
 name Eng
vlan 20
 name Sales
vlan 199
 name Native
 ```
### SW2 (VTP Transparent + Router Link)
```
vtp mode transparent
vtp domain Flackbox

interface gig0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 199
interface gig0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 199

interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk

vlan 10
 name Eng
vlan 20
 name Sales
vlan 199
 name Native
 ```
### SW3 (VTP Client)
```
vtp mode client
vtp domain Flackbox

interface gig0/2
 switchport mode trunk
 switchport trunk native vlan 199

interface range f0/1 - 2
 switchport mode access
 switchport access vlan 20
interface f0/3
 switchport mode access
 switchport access vlan 10
 ```

 ### R1 – Router-on-a-Stick
 ```
 interface FastEthernet0/0
 no ip address
 no shutdown

interface FastEthernet0/0.10
 encapsulation dot1q 10
 ip address 10.10.10.1 255.255.255.0

interface FastEthernet0/0.20
 encapsulation dot1q 20
 ip address 10.10.20.1 255.255.255.0
```
 ### Verification Results Intra-VLAN Connectivity (Before Routing)
 ```
 ENG1 → ENG3  (10.10.10.10 → 10.10.10.12)  → Success
 SALES1 → SALES3 (10.10.20.10 → 10.10.20.12) → Success
```

### Inter-VLAN Connectivity (After Router-on-a-Stick)
```
ENG1 → SALES1 (10.10.10.10 → 10.10.20.10)
First packet timeout (proxy ARP), then 3/4 replies → Success
ENG1 → Sales gateway (10.10.20.1) → 100% success
```

### Key Takeaways & Best Practices Applied

Native VLAN changed from 1 → 199 on all trunks (prevents VLAN hopping)
SW2 in Transparent mode → immune to VTP bombs
SW3 learns VLANs automatically via VTP Client
Router-on-a-Stick uses only one physical link to R1 (efficient for small setups)
Consistent IP scheme: 10.10.[VLAN].1 as gateway
