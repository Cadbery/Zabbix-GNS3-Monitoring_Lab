# Enterprise Network Architecture Simulation & NOC Monitoring Lab

A simulated enterprise network environment designed in **GNS3**, featuring **VyOS** inter-VLAN routing, **Open vSwitch (OVS)** VLAN segmentation, and automated **Zabbix NOC** monitoring with Chaos Incident Response testing.

---

## Executive Summary

The goal of this project is to build and validate a resilient enterprise network architecture with proactive Network Operations Center (NOC) monitoring. By implementing 802.1Q VLAN segmentation, stateful firewall rules, and agentless HTTP/ICMP polling, the environment provides real-time visibility into network health and application availability.

To prove the efficacy of the monitoring setup, five **Chaos Matrix** failure scenarios were executed to test alert triggers for Layer 2/3 link failures, Layer 7 service outages, firewall policy blocks, and server performance degradation.

---

## Lab Deployment & Setup

1. **Topology Deployment:** Imported node appliances (VyOS, OVS, Ubuntu Server, VPCS) into GNS3.
2. **Layer 2/3 Provisioning:** Configured OVS VLAN access/trunk tags and VyOS 802.1Q sub-interfaces (`eth1.10`, `eth1.20`, `eth1.99`).
3. **Services:** Deployed Nginx container on VLAN 20 and Zabbix Server/Frontend stack on VLAN 99.
4. **Monitoring Rules:** Configured ICMP ping triggers and agentless HTTP Web Scenarios for application SLA tracking.

> *Note: Complete device configurations and exported Zabbix scenarios are available in the [`configs/`](./configs/) directory.*

---

## Network Architecture & Topology

The topology separates traffic across three primary VLANs routed through a central VyOS gateway and switched via Open vSwitch:

                                              +--------------------------+
                                              |    VyOS Core Router      |
                                              |  (eth1.10, .20, .99)     |
                                              +------------+-------------+
                                                           | (Trunk - 802.1Q)
                                              +------------+-------------+
                                              |     Open vSwitch (OVS)   |
                                              +---+--------+---------+---+
                                                  |        |         |
                                     +------------+        |         +------------+
                            (Tag 10) |                     | (Tag 20)             | (Tag 99)
                              +------v-------+      +------v-------+       +------v-------+
                              |  PC 1 Client |      | Web_Server-1 |       |  NOC Server  |
                              |     DHCP     |      | 192.168.20.10|       |192.168.99.100|
                              +--------------+      +--------------+       +--------------+

---

### IP & VLAN Addressing Scheme

| VLAN ID     | Subnet / Network  | Device Name      | Assigned IP      | Switch / Router Port          | Role & Description                     |
| ----------- | ----------------- | ---------------- | ---------------- | ----------------------------- | -------------------------------------- |
| **VLAN 10** | `192.168.10.0/24` | **PC 1**         | `192.168.10.x`   | VyOS `eth1.10` / OVS `tag=10` | Client Workstation Subnet              |
| **VLAN 20** | `192.168.20.0/24` | **Web_Server-1** | `192.168.20.10`  | VyOS `eth1.20` / OVS `tag=20` | DMZ Application Subnet (Nginx HTTP)    |
| **VLAN 99** | `192.168.99.0/24` | **NOC Server**   | `192.168.99.100` | VyOS `eth1.99` / OVS `tag=99` | Management & Zabbix Monitoring         |
| **N/A**     | `192.168.87.0/24` | **VyOS Router**  | `192.168.87.129` | Upstream WAN / Cloud          | Inter-VLAN Routing & Stateful Firewall |

---

## Tech Stack & Tools

* **Virtualization / Hypervisor:** VMware Workstation Pro
* **Network Emulation:** GNS3
* **Routing & Firewall:** VyOS 1.4+ (nftables engine)
* **Switching:** Open vSwitch (`ovs-vsctl`)
* **Monitoring & Alerting:** Zabbix (Agentless Web Scenarios & ICMP Polling)
* **Application Services:** Docker / Nginx Web Server

---

## The Chaos Matrix (Incident Response Testing)

Five chaos tests were conducted to verify Zabbix monitoring visibility across different layers of the OSI model:

### 1. Switch Interface Shutdown (Layer 2)
* **Scenario:** Shut down the access switch interface connecting a key segment.
* **Action:** Disabled interface via Open vSwitch / GNS3 link break.
* **Result:** Zabbix immediately raised an **"Interface Down"** high-severity trigger.
* **Evidence:** `screenshots/test1-interface-down.png`

### 2. Routing Table Sabotage (Layer 3)
* **Scenario:** Disrupted inter-VLAN static/gateway routes on the VyOS router.
* **Action:** Removed or misconfigured sub-interface routes on VyOS.
* **Result:** ICMP poller detected network unreachability, logging downtime duration and packet loss metrics.
* **Evidence:** `screenshots/test2-routing-failure.png`

### 3. HTTP Application Service Crash (Layer 7 vs. Layer 3)
* **Scenario:** Stopped the web daemon while keeping the VM online to test host vs. service monitoring.
* **Action:** Stopped Nginx daemon (`nginx -s stop` / container stop).
* **Result:** Demonstrates the difference between **Host Down** (ICMP) and **Service Down** (HTTP 502/Refused). Zabbix Web Scenario triggered an **"HTTP Service Down"** alert while ICMP ping remained green.
* **Evidence:** `screenshots/test3-service-down.png`

### 4. Firewall / ACL Isolation Test
* **Scenario:** Verified stateful firewall filtering across VLAN boundaries.
* **Action:** Applied a VyOS `forward filter` rule blocking TCP port 80 egress to VLAN 20:
  ```text
  set firewall ipv4 forward filter rule 10 action drop
  set firewall ipv4 forward filter rule 10 protocol tcp
  set firewall ipv4 forward filter rule 10 destination port 80
* **Result:** Proven ability to monitor and alert on firewall/access-control blocking without taking down the underlying host.
* **Evidence:** screenshots/test4-firewall-block.png

### 5. CPU Stress & Load Alerting (Performance SLA)
* **Scenario:** Triggered performance threshold alerts before a server actually crashes.
* **Action:** Generated heavy HTTP/CPU stress load on the web server container.
* **Result:** Captured HTTP homepage check response time spiking from a baseline of ~21.64 ms to a peak of 1.66 seconds (1,661.41 ms), triggering latency degradation alerts.
* **Evidence:** screenshots/test5-cpu-latency-spike.png
