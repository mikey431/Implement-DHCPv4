# Packet Tracer - Implement DHCPv4

## 📌 Project Overview

This repository contains the configuration guide and verification steps for the **Implement DHCPv4** Packet Tracer activity. In this lab, a central Cisco router (`R2`) is configured as a DHCPv4 server to dynamically allocate IPv4 addresses to remote LAN segments (`R1` and `R3`) via DHCP relay agents (`ip helper-address`), while also acting as a DHCPv4 client on its WAN interface (`G0/1`) to obtain dynamic addressing from an ISP.

## 📋 Addressing Table

| Device | Interface | IPv4 Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | `G0/0` | `192.168.10.1` | `255.255.255.0` | N/A |
| **R1** | `S0/0/0` | `10.1.1.1` | `255.255.255.252` | N/A |
| **R2** | `G0/0` | `192.168.20.1` | `255.255.255.0` | N/A |
| **R2** | `G0/1` | *DHCP Assigned* | *DHCP Assigned* | N/A |
| **R2** | `S0/0/0` | `10.1.1.2` | `255.255.255.252` | N/A |
| **R2** | `S0/0/1` | `10.2.2.2` | `255.255.255.252` | N/A |
| **R3** | `G0/0` | `192.168.30.1` | `255.255.255.0` | N/A |
| **R3** | `S0/0/1` | `10.2.2.1` | `255.255.255.252` | N/A |
| **PC1** | `NIC` | *DHCP Assigned* | *DHCP Assigned* | *DHCP Assigned* |
| **PC2** | `NIC` | *DHCP Assigned* | *DHCP Assigned* | *DHCP Assigned* |
| **DNS Server** | `NIC` | `192.168.20.254` | `255.255.255.0` | `192.168.20.1` |

## 🎯 Objectives

* **Part 1:** Configure a Router as a DHCP Server
* **Part 2:** Configure DHCP Relay
* **Part 3:** Configure a Router as a DHCP Client

## ⚙️ Configuration Steps

### Part 1: Configure a Router as a DHCP Server (`R2`)

1. **Configure Excluded IPv4 Addresses:** Reserve the first 10 IP addresses from both the `R1` (`192.168.10.0/24`) and `R3` (`192.168.30.0/24`) LANs to prevent IP conflicts with default gateways and statically assigned hosts.
2. **Create DHCP Pools:** Create case-sensitive DHCP pools (`R1-LAN` and `R3-LAN`) specifying the network subnet, default gateway, and DNS server (`192.168.20.254`).

```cisco
R2> enable
R2# configure terminal

! Step 1: Exclude the first 10 addresses from R1 and R3 LANs
R2(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10
R2(config)# ip dhcp excluded-address 192.168.30.1 192.168.30.10

! Step 2: Create DHCP Pool for R1 LAN
R2(config)# ip dhcp pool R1-LAN
R2(dhcp-config)# network 192.168.10.0 255.255.255.0
R2(dhcp-config)# default-router 192.168.10.1
R2(dhcp-config)# dns-server 192.168.20.254
R2(dhcp-config)# exit

! Step 3: Create DHCP Pool for R3 LAN
R2(config)# ip dhcp pool R3-LAN
R2(dhcp-config)# network 192.168.30.0 255.255.255.0
R2(dhcp-config)# default-router 192.168.30.1
R2(dhcp-config)# dns-server 192.168.20.254
R2(dhcp-config)# exit
```

### Part 2: Configure DHCP Relay (`R1` & `R3`)

Because routers do not forward broadcast traffic by default, `R1` and `R3` must be configured as DHCP relay agents to forward client DHCP broadcasts as unicast packets to `R2`'s directly connected serial interfaces.

#### Step 1: Configure `R1` and `R3` as DHCP Relay Agents

```cisco
! Router R1 Configuration
R1> enable
R1# configure terminal
R1(config)# interface GigabitEthernet 0/0
R1(config-if)# ip helper-address 10.1.1.2
R1(config-if)# exit
```

```cisco
! Router R3 Configuration
R3> enable
R3# configure terminal
R3(config)# interface GigabitEthernet 0/0
R3(config-if)# ip helper-address 10.2.2.2
R3(config-if)# exit
```

#### Step 2: Set `PC1` and `PC2` to Receive IP Addressing from DHCP

* Navigate to **Desktop > IP Configuration** on both **PC1** and **PC2**, and switch the IP configuration from **Static** to **DHCP**.
* **PC1** will dynamically receive `192.168.10.11/24`, and **PC2** will dynamically receive `192.168.30.11/24`.

### Part 3: Configure `R2` as a DHCP Client

Configure interface `GigabitEthernet 0/1` on `R2` to obtain an IPv4 address dynamically from the ISP and activate the interface.

```cisco
R2(config)# interface GigabitEthernet 0/1
R2(config-if)# ip address dhcp
R2(config-if)# no shutdown
R2(config-if)# exit
```

## 🔍 Verification & Testing

1. **Check Active DHCP Leases on `R2`:**
   ```cisco
   R2# show ip dhcp binding
   ```
   *Verifies that `PC1` (`192.168.10.11`) and `PC2` (`192.168.30.11`) have been leased addresses from their respective pools.*

2. **Verify `R2` WAN DHCP Client Interface:**
   ```cisco
   R2# show ip interface brief
   ```
   *Confirms `GigabitEthernet0/1` is `up/up` and assigned an IPv4 address via `DHCP`.*

3. **Test End-to-End Connectivity:**
   * Ping between `PC1` and `PC2`.
   * Ping the `DNS Server` (`192.168.20.254`) and `R2`'s ISP-assigned `G0/1` IP address from both workstations.