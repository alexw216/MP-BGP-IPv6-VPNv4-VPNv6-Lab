# **MP‑BGP IPv6 + VPNv4/VPNv6 Lab**  
### *Full Dual‑Stack MPLS L3VPN with IPv6 Underlay, IPv6 Overlay, and Internet Edge*

This lab implements a complete **service‑provider‑grade MP‑BGP environment** with:

- **IPv4 + IPv6 underlay (OSPFv2 + OSPFv3)**
- **MP‑BGP IPv4/IPv6 unicast**
- **MPLS LDP transport**
- **MP‑BGP VPNv4 + VPNv6 (L3VPN)**
- **Dual‑stack VRFs**
- **IPv6 Internet edge (R1/R2 → ISP1/ISP2)**
- **Traffic engineering (LP, AS‑path prepend, MED)**
- **Route Reflectors (R3/R4)**
- **PE routers (R5–R8)**

All router configurations are included inside the attached CML topology YAML file.

---

# **1. Topology Overview**

The topology follows a classic SP hierarchy:

```
          ISP1          ISP2
           |              |
         R1 (Edge-A)   R2 (Edge-B)
           \            /
            \          /
             R3 ---- R4        ← Route Reflectors
            /  \    /  \
          R5  R6  R7  R8        ← PE Routers (VRFs)
```

- **R1/R2** = Internet edge  
- **R3/R4** = Core Route Reflectors  
- **R5–R8** = PE routers hosting VRFs  
- **OSPFv2 + OSPFv3** provide IPv4/IPv6 underlay  
- **MPLS LDP** runs across the core  
- **MP‑BGP** carries IPv4, IPv6, VPNv4, VPNv6  

---

# **2. Underlay Technologies**

## **2.1 IPv4 Underlay (OSPFv2)**  
- All core links run OSPF area 0  
- Loopback0 addresses are advertised  
- Provides IPv4 reachability for LDP and BGP update‑source

## **2.2 IPv6 Underlay (OSPFv3)**  
- All core links run OSPFv3 area 0  
- Loopback0 IPv6 /128s are advertised  
- Ensures IPv6 next‑hop reachability for MP‑BGP IPv6 AF

## **2.3 MPLS LDP Transport**
- Enabled on all core interfaces (R3–R8)  
- Provides label‑switched forwarding for VPNv4/VPNv6

---

# **3. MP‑BGP Control Plane**

## **3.1 IPv4/IPv6 Unicast AF**
- iBGP full‑mesh replaced by RRs (R3/R4)
- IPv4 AF uses IPv4 neighbors  
- IPv6 AF uses IPv6 neighbors (Loopback0 IPv6)

## **3.2 Route Reflector Design**
R3 and R4 reflect:

- IPv4 unicast  
- IPv6 unicast  
- VPNv4  
- VPNv6  

Clients: R1, R2, R5, R6, R7, R8

---

# **4. MPLS L3VPN (VPNv4 + VPNv6)**

## **4.1 VRF Definition (PE Routers R5–R8)**

Example (CUST‑A):

```
vrf definition CUST-A
 rd 65001:100
 route-target export 65001:100
 route-target import 65001:100
 address-family ipv4
 address-family ipv6
```

## **4.2 VRF Interfaces**

| Router | IPv4 CE Subnet | PE IPv4 | IPv6 CE Subnet | PE IPv6 |
|--------|----------------|---------|----------------|---------|
| R5 | 10.10.5.0/24 | 10.10.5.1 | 2001:DB8:A:5::/64 | 2001:DB8:A:5::1 |
| R6 | 10.10.6.0/24 | 10.10.6.1 | 2001:DB8:A:6::/64 | 2001:DB8:A:6::1 |
| R7 | 10.10.7.0/24 | 10.10.7.1 | 2001:DB8:A:7::/64 | 2001:DB8:A:7::1 |
| R8 | 10.10.8.0/24 | 10.10.8.1 | 2001:DB8:A:8::/64 | 2001:DB8:A:8::1 |

## **4.3 MP‑BGP VRF AFs**

Each PE redistributes connected VRF routes:

```
address-family ipv4 vrf CUST-A
 redistribute connected

address-family ipv6 vrf CUST-A
 redistribute connected
```

---

# **5. IPv6 Internet Edge (R1/R2 → ISP1/ISP2)**

## **5.1 IPv6 Addressing**

Example (R1 ↔ ISP1):

```
R1 Gi0/2:   fd00:1:1::1/64
ISP1 Gi0/0: fd00:1:1::2/64
```

## **5.2 eBGP IPv6 Peering**

```
router bgp 65001
 address-family ipv6
  neighbor fd00:1:1::2 activate
  neighbor fd00:1:1::2 soft-reconfiguration inbound
```

ISP advertises default:

```
network ::/0
```

---

# **6. Traffic Engineering**

## **6.1 Local Preference**
Prefer ISP1:

```
set local-preference 200
```

## **6.2 AS‑Path Prepending**
Make R1 less attractive outbound:

```
set as-path prepend 65001 65001 65001
```

## **6.3 MED**
Used on R2 toward ISP2:

```
set metric 50
```

---

# **7. Advanced MP‑BGP Features**

## **7.1 Graceful Restart**

```
router bgp 65001
 bgp graceful-restart
 timers bgp 30 90
```

## **7.2 BFD (optional)**

```
interface Gi0/x
 bfd interval 50 min_rx 50 multiplier 3

router bgp 65001
 neighbor X fall-over bfd
```

---

# **8. Verification Workbook**

This section provides CCIE‑style verification commands.

---

## **8.1 Underlay Verification**

### IPv4 OSPF
```
show ip ospf neighbor
show ip route ospf
```

### IPv6 OSPFv3
```
show ipv6 ospf neighbor
show ipv6 route ospf
```

---

## **8.2 MPLS Verification**

```
show mpls interfaces
show mpls ldp neighbor
show mpls forwarding-table
```

---

## **8.3 MP‑BGP IPv4/IPv6 Unicast**

```
show bgp ipv4 unicast summary
show bgp ipv6 unicast summary
show bgp ipv6 unicast 2001:DB8:1::7/128
```

---

## **8.4 VPNv4/VPNv6 Control Plane**

```
show bgp vpnv4 all summary
show bgp vpnv6 all summary
show bgp vpnv4 vrf CUST-A
show bgp vpnv6 vrf CUST-A
```

---

## **8.5 VRF Routing**

```
show ip route vrf CUST-A
show ipv6 route vrf CUST-A
```

---

## **8.6 End‑to‑End L3VPN Tests**

### IPv4 VPN ping
```
ping vrf CUST-A 10.10.7.1
```

### IPv6 VPN ping
```
ping vrf CUST-A ipv6 2001:DB8:A:7::1
```

### MPLS‑aware traceroute
```
traceroute vrf CUST-A ipv6 2001:DB8:A:7::1
```

---

## **8.7 Internet Edge Tests**

### Default route from ISP
```
show bgp ipv6 unicast ::/0
```

### IPv6 Internet reachability
```
ping ipv6 fd00:1:1::2 source 2001:DB8:1::1
```

---

# **9. Files Included**

- **`mp-bgp-lab-ipv6-vpn4-vpn6.yaml`**  
  Contains:
  - Full CML topology  
  - All router configurations  
  - Interface mappings  
  - MPLS, OSPF, BGP, VRF configs  

---

# **10. Conclusion**

This lab provides a complete, production‑grade **dual‑stack MPLS L3VPN environment** with:

- IPv6 underlay  
- IPv6 overlay  
- VPNv4/VPNv6  
- Route reflection  
- MPLS transport  
- Internet edge  
- Traffic engineering  

It is suitable for:

- CCNP SP  
- CCIE SP  
- CCIE Enterprise Infrastructure (MPLS/MP‑BGP sections)  
- Real‑world SP architecture practice  