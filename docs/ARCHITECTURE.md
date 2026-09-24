# NalaNetra FloodGrid — Technical Architecture & Engineering Design

This document details the multi-tiered system architecture, scientific mathematical formulation, spatial database modeling, and state machine lifecycle of **NalaNetra FloodGrid**.

---

## 🏛️ 3-Tier System Architecture

NalaNetra is designed as a decoupled, microservices-driven platform structured into three specialized operating tiers:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│               TIER 1: ADMINISTRATION & COMMAND (MCG OFFICER PORTAL)                   │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ • Interactive Leaflet.js GIS Heatmap & Hotspot Feed                                    │
│ • Algorithmic Priority Queue with Officer Verification Gate                            │
│ • Live Crew Roster, Turn-by-Turn Telemetry & Municipal SLA Tracking                    │
└────────────────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼ (REST / WebSocket / GeoJSON)
┌────────────────────────────────────────────────────────────────────────────────────────┐
│               TIER 2: CORE INTELLIGENCE ENGINE (FASTAPI & POSTGIS)                     │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ • Radar Rainfall Ingestion & Optical Flow Nowcasting (IMD Aya Nagar / NCMRWF UM)       │
│ • 1D-Kinematic Wave & Manning's Hydraulic Network Simulation (EPA-SWMM Standard)       │
│ • Multi-Variable Scientific Priority Formula (P-Score Engine)                          │
│ • DBSCAN Spatial Clustering (150m Epsilon / 6-Hour Temporal Window)                    │
│ • Flood-Aware Dynamic Routing Engine (OSRM with Flooded Edge Penalization)             │
│ • Computer Vision Depth Triage & SHA-256 Cryptographic Audit Ledger                    │
└────────────────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼ (REST APIs / Push Notifications)
┌────────────────────────────────────────────────────────────────────────────────────────┐
│               TIER 3: END-USER & FIELD RESPONSE (CITIZEN & CREW APPS)                  │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ • Citizen App: 1-Tap Bilingual (Hindi/English) Reporting, Live 5-Stage Stepper         │
│ • Field Crew App: Turn-by-Turn Safe Navigation, Offline SQLite Queue                   │
│ • Proof Gate: Mandatory Geo-Fenced Before/After Photo & EXIF Verification              │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Mathematical & Hydrological Formulations

### 1. Scientific Priority Formula
Every reported hotspot is scored using a multi-variable index bounded between $0.00$ and $1.00$ (scaled to 100):

$$\mathbf{P = 0.30(S) + 0.20(R) + 0.15(W) + 0.15(D) + 0.10(E) + 0.10(A)}$$

- **$\mathbf{S}$ (0.30): Ground Truth Depth Tier**  
  Ankle $= 0.25$ ($<15\text{ cm}$), Knee $= 0.50$ ($15\text{--}40\text{ cm}$), Waist $= 0.75$ ($40\text{--}80\text{ cm}$), Submerged $= 1.00$ ($>80\text{ cm}$ or vehicle floating).
- **$\mathbf{R}$ (0.20): Doppler Radar Rain Nowcast Intensity**  
  Normalized rain intensity $R / 80\text{ mm/hr}$, derived from radar reflectivity $Z = 200 R^{1.6}$.
- **$\mathbf{W}$ (0.15): Ward Criticality Factor**  
  Static weight assigned by Municipal Corporation Gurugram based on ward population density, commercial importance, and vulnerable slum clusters.
- **$\mathbf{D}$ (0.15): Drainage Pipe Surcharge Index**  
  Ratio of upstream hydraulic load to maximum conduit capacity ($Q / Q_{cap}$). Surcharged when $\ge 1.0$.
- **$\mathbf{E}$ (0.10): Emergency Route Corridor Multiplier**  
  $1.5\times$ weight factor applied if the incident lies within a 50m buffer of an arterial ambulance corridor (e.g., Medanta / Civil Hospital transit routes).
- **$\mathbf{A}$ (0.10): SLA Aging Factor**  
  Linear escalation: $A = \min(1.0, \Delta t / 180\text{ min})$. Ensures stagnant unaddressed incidents rise to the top of the queue.

---

### 2. Manning's Equation for Pipe Surcharge
Conduit discharge capacity is computed using standard open-channel and gravity pipe hydraulics:

$$\mathbf{Q_{cap} = A \cdot \frac{1}{n} \cdot R_h^{2/3} \cdot S^{1/2}}$$

- $A$: Cross-sectional area of the conduit ($m^2$)
- $n$: Manning's roughness coefficient ($0.013$ for reinforced concrete pipes)
- $R_h$: Hydraulic radius ($A / P_w$, where $P_w$ is wetted perimeter)
- $S$: Bed slope of the stormwater pipe ($\Delta z / L$)

---

### 3. Spatial Incident Clustering (DBSCAN)
To prevent duplicate crew dispatch when multiple citizens report the same flooded underpass:

$$\mathbf{N_\varepsilon(p) = \{q \in D \mid \text{dist}(p, q) \le \varepsilon \quad \text{and} \quad |t_p - t_q| \le \Delta t\}}$$

- $\varepsilon = 150\text{ meters}$ (Spatial threshold)
- $\Delta t = 6\text{ hours}$ (Temporal event window)
- MinPts $= 1$ (Forms clusters dynamically, linking subsequent reports to the master ticket)

---

## 🔄 Proof-Gated Incident State Machine

```
   [Citizen Reports Waterlogging]
                │
                ▼
   [AI Validity & Spam Screening] ──(Fake / Ineligible)──► [Rejected]
                │ (Valid)
                ▼
   [150m Spatial Clustering Engine]
                │
                ▼
   [P-Score Calculated & Queued]
                │
                ▼
   [Officer Verification & Dispatch]
                │
                ▼
   [Field Crew Dispatched via OSRM Safe Nav]
                │
                ▼
   [On-Site Work & Drain Cleared]
                │
                ▼
   [Mandatory Proof Gate] ◄── Geotagged After-Photo within 50m Geo-Fence
                │
                ▼
   [SHA-256 Hash Locked & Ticket Closed] ──► [Citizen Notified via Stepper]
```
