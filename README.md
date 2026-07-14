# 🌐📞 The TELL System — Multi-Site Enterprise VoIP Network (Cisco CME + Full-Mesh OSPF)

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![OSPF](https://img.shields.io/badge/Routing-OSPF%20Mesh-blue?style=for-the-badge)
![VoIP](https://img.shields.io/badge/Voice-CME%20%2F%20VoIP-green?style=for-the-badge)
![Sites](https://img.shields.io/badge/Sites-4%20Independent%20CME-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

> A **4-site enterprise telephony network** — Branch-A, Branch-B, Branch-C, and a Public-Side gateway — each running an independent Cisco Unified CallManager Express (CME) instance, fully interconnected over a **redundant OSPF mesh backbone**, enabling any extension at any site to call any other site.

---

## 📌 Project Overview

**The TELL System** simulates a real-world multi-branch enterprise deploying its own internal VoIP telephony across several physical locations. Rather than a single centralized call manager, **each site runs its own local CME instance** (a distributed voice architecture) while a **full OSPF mesh** stitches every site's data and voice subnets together — allowing seamless inter-site dialing across the entire organization.

The build also includes a **Public-Side site**, representing an external/gateway location, proving that the internal dial plan can extend beyond the core branches.

Built and fully tested in **Cisco Packet Tracer**.

---

## 🗺️ Network Topology

```
                                  ┌───────────────────────┐
                                  │   PUBLIC-SIDE (R3)     │
                                  │  Switch3 → PC15 + IP   │
                                  │  Phone15 (Ext. 400)    │
                                  │  Voice Net: 192.168.8. │
                                  └───────────┬────────────┘
                                              │ Serial
                                              │
                                       ┌──────┴──────┐
                        ┌──────────────┤  Router0     ├──────────────┐
                        │  (mesh)      │ (Core)       │   (mesh)     │
                        │              └──────────────┘              │
                 ┌──────┴──────┐                             ┌───────┴──────┐
                 │  Router2     │◄──────── mesh link ───────►│  Router1      │
                 │ BRANCH-A     │                             │  (hub)        │
                 │ TELL-100     │                             └───────┬───────┘
                 └──────┬───────┘                                     │
                        │                                    ┌────────┴────────┐
              ┌─────────┴─────────┐                          │                 │
              │ Switch0 (Branch-A)│                    Switch1 (BRANCH-B) Switch2 (BRANCH-C)
              │ ROOM 1–5          │                    TELL-200           TELL-300
              │ PC0-4 + Phones0-4 │                    ROOM A-E           ROOM I-V
              │ Ext. 101–104      │                    PC5-9+Phones5-9    PC10-14+Phones10-14
              └───────────────────┘                    Ext. 201–205       Ext. 301–305
```

| Site | Router | Role | Voice Subnet | Extensions | Devices |
|---|---|---|---|---|---|
| **Branch-A** | Router2 | CME (TELL-100) | `192.168.2.0/24` | 101 – 104 | Rooms 1–5 (PC0–4, IP Phone0–4) |
| **Branch-B** | Router1 | CME (TELL-200) | `192.168.4.0/24` | 201 – 205 | Rooms A–E (PC5–9, IP Phone5–9) |
| **Branch-C** | Router0* | CME (TELL-300) | `192.168.6.0/24` | 301 – 305 | Rooms I–V (PC10–14, IP Phone10–14) |
| **Public-Side** | Router3 | CME Gateway | `192.168.8.0/24` | 400 | PC15 + IP Phone15 |

\* Site labeling verified against each router's `telephony-service ip source-address`.

**Backbone:** All routers interconnect via serial WAN links running **OSPF Area 0**, with a partial full-mesh (Router0 ↔ Router1 ↔ Router2 ↔ Router3) providing redundant paths between sites.

---

## ⚙️ Technologies & Concepts Used

- ✅ **Multi-Site OSPF Mesh** — redundant serial links between all core routers, single Area 0
- ✅ **Distributed Cisco CME Architecture** — each site runs its own `telephony-service` process instead of one centralized call manager
- ✅ **Structured Extension Numbering Plan** — 1xx / 2xx / 3xx / 4xx blocks per site for clean dial-plan routing
- ✅ **Inter-Site VoIP Dial-Peers** — every router configured with dial-peers matching every other site's extension range
- ✅ **Router-on-a-Stick / VLAN segmentation** per branch (Data + Voice)
- ✅ **Cisco 7960 IP Phones** — SCCP-registered endpoints across all 4 sites
- ✅ **Public-Side Gateway Simulation** — an external-facing site fully integrated into the internal dial plan

---

## 🔧 Key Configuration Highlights

**1. Site-Specific CME (Branch-A / Router2 — TELL-100)**
```
telephony-service
 max-ephones 10
 max-dn 10
 ip source-address 192.168.2.1 port 2000
 auto assign 1 to 10

ephone-dn 1
 number 101
ephone-dn 2
 number 102
ephone-dn 3
 number 103
ephone-dn 4
 number 104
```

**2. Inter-Site Dial-Peers (Branch-A → Branch-B, Branch-C, Public-Side)**
```
dial-peer voice 1 voip
 destination-pattern 2..
 session target ipv4:192.168.4.1
!
dial-peer voice 2 voip
 destination-pattern 3..
 session target ipv4:192.168.6.1
!
dial-peer voice 3 voip
 destination-pattern 4..
 session target ipv4:192.168.8.1
```

**3. Public-Side Gateway (Router3)**
```
telephony-service
 max-ephones 10
 max-dn 10
 ip source-address 192.168.8.1 port 2000
 auto assign 1 to 10

ephone-dn 1
 number 400

dial-peer voice 3 voip
 destination-pattern 1..
 session target ipv4:192.168.2.1
dial-peer voice 5 voip
 destination-pattern 2..
 session target ipv4:192.168.4.1
dial-peer voice 6 voip
 destination-pattern 3..
 session target ipv4:192.168.6.1
```

**4. OSPF Backbone (sample — full mesh routing)**
```
router ospf 1
 network 10.0.0.0 0.255.255.255 area 0
 network 20.0.0.0 0.255.255.255 area 0
 network 30.0.0.0 0.255.255.255 area 0
 network 192.168.0.0 0.0.255.255 area 0
```

---

## 🧪 Testing & Verification

| Test | Method | Result |
|---|---|---|
| OSPF Route Propagation | `show ip route` on Router0/Router1/Router2/Router3 | ✅ All 4 sites' data & voice subnets learned across the mesh via OSPF (`O` routes) |
| Backbone Redundancy | Mesh serial links (10.x, 20.x, 30.x) | ✅ Multiple OSPF paths visible per subnet (e.g., dual paths to 30.1.1.0/24) |
| Branch-A → Public-Side Call | Dial `400` from IP Phone2 (Ext. 101) | ✅ `Ring Out` → **Connected** on both IP Phone2 and IP Phone15 |
| Branch-B → Branch-A Call | Dial `101` from IP Phone9 (Ext. 204) | ✅ IP Phone2 shows **"The phone is ringing" / From: 204**, IP Phone9 shows **Ring Out** |
| Phone Registration | `show ephone` per site | ✅ All ephones REGISTERED with correct MAC/IP bindings per branch |

### 📸 Evidence Snapshots
- Full CLI dial-peer & telephony-service configs across **Router0, Router1, Router2, Router3**
- `show ip route` output confirming complete OSPF mesh convergence (all four sites' subnets visible from any router)
- Live call: **Ext. 101 (Branch-A) → Ext. 400 (Public-Side)** — Ring Out → Connected
- Live call: **Ext. 204 (Branch-B) → Ext. 101 (Branch-A)** — Ringing → Ring Out
- Full topology diagram showing all 4 sites, VLAN color-coding, and mesh WAN links

---

## 🎯 Key Outcomes

- Delivered a **fully distributed, 4-site VoIP architecture** — no single point of call-processing failure, unlike a centralized CallManager design
- Achieved **complete OSPF mesh convergence** with redundant paths between core sites
- Designed and implemented a **clean, scalable extension-numbering dial plan** (1xx/2xx/3xx/4xx) that simplified dial-peer configuration across all sites
- Proved **successful cross-site calling** in both directions (Branch-A↔Public-Side, Branch-B↔Branch-A), validating dial-peer and routing integration end-to-end
- Simulated a **Public-Side gateway site**, demonstrating how an external-facing location integrates cleanly into an internal enterprise dial plan

---

## 🚀 Skills Demonstrated

`Multi-Site OSPF Design` · `Cisco Unified CME` · `VoIP Dial-Plan Engineering` · `Redundant WAN Mesh Routing` · `VLAN Segmentation` · `Cisco IP Telephony` · `Network Troubleshooting` · `Enterprise Voice Architecture`

---

## 👤 Author

**Maaz Khan**
CCNA-Certified Network & NOC Engineer | Cisco ID: CSCO15140445
📍 Lower Dir, KPK, Pakistan

Part of an ongoing series of 100+ Cisco Packet Tracer lab projects documented on GitHub & LinkedIn — this build extends the earlier two-branch CME lab into a full 4-site distributed voice mesh.

---

⭐ If you found this project useful, consider giving the repo a star!
