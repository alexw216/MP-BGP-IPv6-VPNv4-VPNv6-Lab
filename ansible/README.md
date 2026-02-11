# **README.md — Ansible Automation for MP‑BGP IPv6 + VPNv4/VPNv6 Lab**

This repository contains a complete **Ansible‑driven automation framework** for deploying a full **dual‑stack MPLS L3VPN service provider lab** based on Cisco IOSv routers.

The lab topology and base configurations originate from a CML YAML file and have been converted into a fully declarative, reproducible Ansible structure.

The automation supports:

- IPv4 + IPv6 underlay (OSPFv2 + OSPFv3)
- MPLS LDP transport
- MP‑BGP IPv4/IPv6 unicast
- MP‑BGP VPNv4/VPNv6 (L3VPN)
- VRFs (dual‑stack)
- Route Reflectors (R3/R4)
- PE routers (R5–R8)
- IPv6 Internet edge (R1/R2 → R9/R10)
- ISP routers (R9/R10)
- Full configuration generation via Jinja2
- Automated deployment, verification, and backup

---

# **1. Directory Structure**

```
ansible/
├── inventory/
│   └── hosts.yml
├── group_vars/
│   ├── all.yml
│   ├── edge.yml
│   ├── rr.yml
│   ├── pe.yml
│   ├── core.yml
│   └── isp.yml
├── host_vars/
│   ├── R1.yml
│   ├── R2.yml
│   ├── R3.yml
│   ├── R4.yml
│   ├── R5.yml
│   ├── R6.yml
│   ├── R7.yml
│   ├── R8.yml
│   ├── R9.yml
│   └── R10.yml
├── templates/
│   └── full_config.j2
├── playbooks/
│   ├── deploy.yml
│   ├── verify.yml
│   └── backup.yml
└── README.md
```

---

# **2. Inventory**

The inventory defines all routers and their functional groups:

- **edge** → R1, R2  
- **rr** → R3, R4  
- **pe** → R5–R8  
- **core** → R3–R8  
- **isp** → R9, R10  

Credentials and connection parameters are defined globally.

See: `inventory/hosts.yml`

---

# **3. group_vars**

Group variables define shared behavior:

| File | Purpose |
|------|---------|
| `all.yml` | Global ASN, OSPF, OSPFv3, MPLS, VRFs |
| `edge.yml` | Marks R1/R2 as edge routers |
| `rr.yml` | Route reflector client lists |
| `pe.yml` | Enables VPNv4/VPNv6 AFs |
| `core.yml` | Enables MPLS LDP |
| `isp.yml` | Defines ISP role (eBGP only) |

---

# **4. host_vars**

Each router has a dedicated `host_vars/<router>.yml` file containing:

- Loopback IPv4/IPv6  
- Interface definitions  
- OSPF/OSPFv3 flags  
- MPLS flags  
- BGP neighbors (IPv4/IPv6)  
- VRF assignments (PE routers)  
- ISP ASNs (R9/R10)

These files are **1:1 translations** of the configs inside the original CML YAML.

---

# **5. Template**

The `full_config.j2` template renders the complete IOS configuration for each router.

It dynamically builds:

- Interfaces  
- OSPFv2  
- OSPFv3  
- MPLS LDP  
- BGP IPv4/IPv6  
- VPNv4/VPNv6  
- VRFs  
- ISP eBGP  
- Route reflector logic  
- PE logic  

The template is role‑aware:

| Role | Behavior |
|------|----------|
| `edge` | iBGP + eBGP (ISP) |
| `rr` | Reflect IPv4/IPv6/VPNv4/VPNv6 |
| `pe` | VRFs + VPNv4/VPNv6 |
| `core` | MPLS LDP |
| `isp` | eBGP only |

---

# **6. Playbooks**

## **6.1 deploy.yml**

Renders configs and pushes them to devices:

```
ansible-playbook playbooks/deploy.yml
```

This:

1. Renders `full_config.j2` into `configs/<router>.cfg`
2. Pushes the config to each router
3. Saves the config if modified

---

## **6.2 verify.yml**

Runs control‑plane checks:

- BGP IPv4/IPv6 summary  
- VPNv4/VPNv6 summary (PE/RR only)  

```
ansible-playbook playbooks/verify.yml
```

---

## **6.3 backup.yml**

Collects and stores running configs:

```
ansible-playbook playbooks/backup.yml
```

Backups are saved under:

```
ansible/backups/<router>.cfg
```

---

# **7. Deployment Workflow**

### **Step 1 — Install required collections**

```
ansible-galaxy collection install cisco.ios ansible.netcommon
```

### **Step 2 — Adjust credentials**

Edit:

```
inventory/hosts.yml
```

### **Step 3 — Deploy configs**

```
ansible-playbook playbooks/deploy.yml
```

### **Step 4 — Verify control-plane**

```
ansible-playbook playbooks/verify.yml
```

### **Step 5 — Backup configs**

```
ansible-playbook playbooks/backup.yml
```

---

# **8. Supported Features**

This automation framework supports:

### **Underlay**
- IPv4 OSPFv2
- IPv6 OSPFv3
- Loopback reachability

### **Transport**
- MPLS LDP
- Label switching across R3–R8

### **Overlay**
- MP‑BGP IPv4 unicast
- MP‑BGP IPv6 unicast
- MP‑BGP VPNv4
- MP‑BGP VPNv6

### **Services**
- VRF CUST‑A (dual‑stack)
- CE connectivity on R5–R8

### **Internet Edge**
- R1 ↔ R9 (ISP1)
- R2 ↔ R10 (ISP2)
- IPv6 default route injection (::/0)

---

# **9. Extending the Framework**

You can easily extend this automation to support:

- Additional VRFs  
- Additional PEs  
- EVPN/VXLAN  
- Segment Routing  
- Traffic engineering policies  
- CI/CD pipelines (GitHub Actions)  

The structure is intentionally modular and scalable.

---

# **10. License**

This repository is intended for lab, training, and educational use.

