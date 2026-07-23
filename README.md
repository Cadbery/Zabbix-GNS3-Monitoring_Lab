# Enterprise Network Architecture Simulation & NOC Monitoring Lab

A simulated enterprise network environment designed in **GNS3**, featuring **VyOS** inter-VLAN routing, **Open vSwitch (OVS)** VLAN segmentation, and automated **Zabbix NOC** monitoring with Chaos Incident Response testing.

---

## Executive Summary

The goal of this project is to build and validate a resilient enterprise network architecture with proactive Network Operations Center (NOC) monitoring. By implementing 802.1Q VLAN segmentation, stateful firewall rules, and agentless HTTP/ICMP polling, the environment provides real-time visibility into network health and application availability.

To prove the efficacy of the monitoring setup, five **Chaos Matrix** failure scenarios were executed to test alert triggers for Layer 2/3 link failures, Layer 7 service outages, firewall policy blocks, and server performance degradation.

---

## Network Architecture & Topology

The topology separates traffic across three primary VLANs routed through a central VyOS gateway and switched via Open vSwitch:

