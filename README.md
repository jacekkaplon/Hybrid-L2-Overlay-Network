
# Hybrid Layer-2 Overlay Network for Residential FTTH Infrastructure

A production-grade encrypted Layer-2 overlay network architecture bridging a public cloud VPS node with a residential Proxmox VE hypervisor backend. This deployment engineering solves critical perimeter edge constraints—including carrier Carrier-Grade NAT (CGNAT), dynamic IP allocations, and ISP ingress filtering—allowing secure remote service exposure without disclosing residential public routing endpoints.

<br>
<br>
<br>

## 🛠️ Infrastructure Architecture & Multi-Layer Topology

```text
+---------------------------------------------------------------+
|                        PHYSICAL LAYER                         |
|                                                               |
|    Proxmox Host ---------------- Internet ---------------- VPS|
+---------------------------------------------------------------+
                               │
                               ▼
+---------------------------------------------------------------+
|                         L3 TRANSPORT                          |
|                                                               |
|          WireGuard Tunnel (UDP/51820, 10.77.77.x/24)          |
|    Proxmox: 10.77.77.x <==== encrypted ====> VPS: 10.77.77.x  |
+---------------------------------------------------------------+
                               │
                               ▼
+---------------------------------------------------------------+
|                       L2 ENCAPSULATION                        |
|                                                               |
|    gretap-overlay <==== Ethernet-over-WireGuard ====> gretap  |
+---------------------------------------------------------------+
                               │
                               ▼
+---------------------------------------------------------------+
|                      L2 BROADCAST DOMAIN                      |
|                                                               |
|    vmbr-overlay <========== BRIDGE ==========> br-overlay     |
|    Proxmox: 10.50.0.x/24                  VPS: 10.50.0.x/24   |
+---------------------------------------------------------------+
                               │
                               ▼
+---------------------------------------------------------------+
|                       VIRTUAL MACHINES                        |
|                                                               |
|    VM / LXC Containers (10.50.0.x) behave as if local to VPS  |
+---------------------------------------------------------------+
```
<br>
<br>
<br>

## 🛠️ Core Infrastructure Architecture

The platform establishes a secure Layer-2 broadcast domain over a public Layer-3 WAN by encapsulating **GRETAP (Generic Routing Encapsulation Ethernet Tap)** frames directly inside an hardened **WireGuard VPN** tunnel transport layer. 

* 🌐 **Cloud Gateway Plane:** Public-facing VPS acting as a stateless ingestion proxy, routing incoming traffic securely through the encrypted overlay topology down to the internal network layers.

<br>

* 🔒 **Residential Edge Plane:** A multi-homed Linux routing environment running systemd-networkd/Netplan pipelines to safely ingest and map encapsulated virtual Ethernet interfaces without interrupting primary broadband default gateways.

<br>

* 🔌 **Reverse-VPN Broker Node:** Hardware-level integration of embedded Linux endpoints (Enigma2-based platforms) utilizing persistent reverse tunneling strategies to bypass remote firewall restrictions without requiring native port-forwarding structures on local routers.

<br>
<br>
<br>

---

<br>

## 📋 Engineering Challenges & Core Resolutions

<br>

* ⚡ **MTU Tuning & Frame Fragmentation Mitigation** – Standard WAN frames default to 1500 bytes. Encapsulating Ethernet frames inside GRETAP (+24 bytes) and WireGuard (+60 bytes) introduces significant protocol overhead, leading to severe packet drops and broken TCP handshakes. This engineering deployment successfully isolates and resolves this via prescriptive clamping using a calculated transport MTU of **1420 bytes**, eliminating fragmentation loops.

<br>

* ⚙️ **Netplan & Systemd-Networkd Integration Constraints** – Standard Debian/Ubuntu network initialization routines often suffer from asynchronous race conditions where virtual overlay tunnels attempt to bind before physical interfaces attain an `UP` operational state. The topology resolves this via explicit interface ordering configurations, guaranteeing rock-solid tunnel persistence across system reboots.

<br>

* 🔄 **Asymmetric Routing Loop Elimination** – Direct state traffic entering the cloud VPS boundary risked routing loops when replies attempted to exit through the local residential ISP interface instead of returning back via the tunnel gateway. Implemented precise custom `iptables` Network Address Translation (NAT) structures combined with policy-based routing to enforce strict return path symmetric symmetry.

<br>
<br>
<br>

---

<br>

## 🔬 Project Documentation & Engineering Reports

The entire planning, deployment matrix, and troubleshooting process has been documented across three independent engineering blueprints. You can access the technical PDF reports directly within this repository:

<br>

### 📄 1. Core Overlay Engineering Architecture Blueprint
* **Description:** The foundational design specification for the L2 over L3 deployment. Contains extensive configuration definitions, MTU analysis, transport headers decomposition, and deployment guidelines for FTTH residential perimeters.
<br>

* 👉 [Download Core Architecture Report (PDF)](./documentation/Hybrid%20L2%20Overlay%20Network%20for%20Residential%20FTTH%20using%20WireGuard.pdf)

<br>

### 📄 2. Edge Node Secure Reverse VPN Specification
* **Description:** Implementation deployment guidelines for extending the secure network overlay onto resource-constrained legacy or embedded Linux environments (Octagon architectures) to ensure stateless command-and-control access over restrictive double-NAT topologies.
<br>

* 👉 [Download Reverse VPN Report (PDF)](./documentation/Secure_Reverse_VPN_Project_2026.pdf)

<br>

### 📄 3. Network Verification & Advanced Troubleshooting Log
* **Description:** Real-world incident log documenting network interface edge-cases, systemd race-condition diagnostic traces, asymmetric policy routing verification, and final link stability sign-off validation protocols.
<br>

* 👉 [Download Troubleshooting & Verification Report (PDF)](./documentation/overlay-project.pdf)

<br>
<br>
<br>

---

<br>

*Disclaimer: This repository is part of a secure home-laboratory project used exclusively for advanced network engineering, L2/L3 overlay research, and secure transport architecture prototyping.*
