# Small Enterprise Network Design & Configuration

## 📌 Project Overview

This project is a small enterprise network designed and configured using **Cisco Packet Tracer**.

The goal of the project was to build a segmented and functional network that supports wired and wireless devices while providing automatic IP address assignment, Inter-VLAN communication, and basic network security.

The network includes multiple VLANs for different departments, DHCP services, Router-on-a-Stick Inter-VLAN Routing, a wireless Guest network, and an ISP/Cloud connection.

---

## 🎯 Project Objectives

* Design a small enterprise network topology.
* Segment the network using VLANs.
* Configure access and trunk ports.
* Implement Inter-VLAN Routing using Router-on-a-Stick.
* Configure DHCP for automatic IP address assignment.
* Provide wireless connectivity through an Access Point.
* Isolate the Guest network from internal networks using ACLs.
* Connect the internal network to an ISP router and Cloud.
* Test and troubleshoot network connectivity.

---

## 🗺️ Network Structure

The network is divided into three main VLANs:

| VLAN | Name  | Purpose                | Network         |
| ---- | ----- | ---------------------- | --------------- |
| 10   | STAFF | Staff PCs and Printer  | 192.168.10.0/24 |
| 20   | Admin | Administrative PCs     | 192.168.20.0/24 |
| 30   | Guest | Wireless Guest Devices | 192.168.30.0/24 |

### Default Gateways

| VLAN    | Default Gateway |
| ------- | --------------- |
| VLAN 10 | 192.168.10.1    |
| VLAN 20 | 192.168.20.1    |
| VLAN 30 | 192.168.30.1    |

---

## 🔧 VLAN Configuration

Three VLANs were created on the switch:

```cisco
vlan 10
name STAFF

vlan 20
name Admin

vlan 30
name Guest
```

End devices were assigned to the appropriate VLAN using access ports.

Example:

```cisco
interface fa0/2
switchport mode access
switchport access vlan 10
```

This configures the port as an access port and assigns it to VLAN 10.

---

## 🔗 Trunk Configuration

The connection between the switch and the main router was configured as a trunk.

```cisco
interface fa0/1
switchport mode trunk
```

The trunk allows traffic from multiple VLANs to travel between the switch and router through a single physical connection.

---

## 🌐 Router-on-a-Stick

Router-on-a-Stick was implemented to provide Inter-VLAN Routing.

### VLAN 10 – STAFF

```cisco
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```

### VLAN 20 – Admin

```cisco
interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
```

### VLAN 30 – Guest

```cisco
interface g0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
```

Each subinterface acts as the default gateway for its corresponding VLAN.

---

## 📡 DHCP Configuration

DHCP was configured on the router to automatically assign IP addresses to devices in each VLAN.

### STAFF DHCP

```cisco
ip dhcp pool STAFF
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
```

### Admin DHCP

```cisco
ip dhcp pool ADMIN
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
```

### Guest DHCP

```cisco
ip dhcp pool GUEST
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
```

This allows devices to automatically receive:

* IP Address
* Subnet Mask
* Default Gateway

---

## 📶 Wireless Guest Network

An Access Point was connected to the switch and assigned to **VLAN 30 – Guest**.

The wireless devices connect through the Access Point and receive their IP addresses from the Guest DHCP pool.

Example:

```text
Guest Device
     ↓
Access Point
     ↓
VLAN 30
     ↓
DHCP
     ↓
192.168.30.x
```

---

## 🔐 Guest Network Isolation

The Guest network was designed to be isolated from the internal Staff and Admin networks.

An ACL was configured to prevent Guest devices from accessing:

```text
192.168.10.0/24 → STAFF
192.168.20.0/24 → Admin
```

Example:

```cisco
access-list 100 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255

access-list 100 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255

access-list 100 permit ip any any
```

The ACL is applied to the Guest subinterface:

```cisco
interface g0/0.30
ip access-group 100 in
```

This prevents Guest devices from directly accessing the internal Staff and Admin networks while allowing other permitted traffic.

---

## 🌍 ISP & Cloud Connectivity

The internal network is connected to an ISP router, which is connected to a Cloud device in Cisco Packet Tracer.

Topology:

```text
Cloud
  |
ISP Router
  |
Main Router
  |
Switch
  |
--------------------------------
|              |               |
VLAN 10      VLAN 20         VLAN 30
STAFF        Admin           Guest
|              |               |
PCs          PCs             Wi-Fi
Printer
```

---

## 🧪 Testing & Verification

The network was tested using several Cisco IOS commands and Packet Tracer tools.

### VLAN Verification

```cisco
show vlan brief
```

Used to verify VLAN creation and port assignments.

### Trunk Verification

```cisco
show interfaces trunk
```

Used to verify the trunk connection between the switch and router.

### Interface Verification

```cisco
show ip interface brief
```

Used to verify the status and IP addresses of router interfaces and subinterfaces.

### DHCP Verification

```cisco
show ip dhcp pool
```

Used to verify DHCP pools and address allocation.

### Connectivity Testing

`ping` was used to test communication between devices and VLANs.

Examples:

```text
STAFF → Admin
STAFF → Guest
Guest → STAFF
Guest → Admin
```

Guest-to-internal network communication was tested after implementing the ACL.

---

## 🛠️ Troubleshooting

During the project, a DHCP issue was identified with VLAN 10.

The VLAN and switch configuration were correct, but devices in VLAN 10 were unable to obtain an IP address because a DHCP pool for the STAFF network had not been configured.

The issue was resolved by creating the following DHCP pool:

```cisco
ip dhcp pool STAFF
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
```

After configuring the DHCP pool, STAFF devices successfully received IP addresses automatically.

This troubleshooting process helped verify the relationship between VLAN configuration, default gateways, and DHCP services.

---

## 📁 Project Files

```text
small-enterprise-network-design/
│
├── README.md
│
├── Packet-Tracer/
│   └── small-enterprise-network.pkt
│
├── Screenshots/
│   ├── network-topology.png
│   ├── vlan-configuration.png
│   ├── dhcp-configuration.png
│   ├── router-configuration.png
│   └── connectivity-tests.png
│
└── Documentation/
    └── network-configuration.txt
```

---

## 🧰 Technologies & Tools

* Cisco Packet Tracer
* Cisco IOS
* VLAN
* DHCP
* Inter-VLAN Routing
* Router-on-a-Stick
* 802.1Q Trunking
* Access Ports
* Access Control Lists (ACL)
* Wireless Networking
* Network Troubleshooting

---

## 📚 Learning Outcomes

Through this project, I practiced:

* Designing a small enterprise network.
* Creating and managing VLANs.
* Configuring Cisco switches and routers.
* Understanding trunk and access ports.
* Implementing Inter-VLAN Routing.
* Configuring DHCP services.
* Configuring wireless network connectivity.
* Applying basic network security using ACLs.
* Troubleshooting DHCP and connectivity issues.
* Verifying network configurations using Cisco IOS commands.

---

## 📌 Project Type

**Personal Networking Project**

**Designed and configured using Cisco Packet Tracer.**
