# AWS Q2 – Custom VPC Network Architecture

## 1. Project Overview

This project implements a complete AWS network architecture for a web application using a custom Amazon VPC.

The architecture uses two Availability Zones, public and private subnets, an Internet Gateway, a NAT Gateway, route tables, and a Security Group.

The purpose is to demonstrate network isolation, subnet segmentation, routing, internet connectivity, and traffic control.

---

## 2. Architecture Diagram

```text
                              INTERNET
                                  │
                                  ▼
                       ┌─────────────────────┐
                       │   Internet Gateway  │
                       │ Q2-Internet-Gateway │
                       └──────────┬──────────┘
                                  │
                 ┌────────────────▼────────────────┐
                 │          Q2-Custom-VPC           │
                 │            10.0.0.0/16           │
                 │                                  │
                 │       ap-south-1a               │
                 │    ┌──────────┴──────────┐       │
                 │    │                     │       │
                 │ Public Subnet       Private Subnet
                 │  10.0.1.0/24        10.0.2.0/24
                 │    │                     │       │
                 │    │                NAT Gateway  │
                 │    │                     │       │
                 │       ap-south-1b               │
                 │    ┌──────────┴──────────┐       │
                 │    │                     │       │
                 │ Public Subnet       Private Subnet
                 │  10.0.3.0/24        10.0.4.0/24
                 │                                  │
                 │      Route Tables                │
                 │      Security Group              │
                 └──────────────────────────────────┘
```

---

## 3. AWS Networking Components

| Component | Configuration | Purpose |
|---|---|---|
| VPC | `Q2-Custom-VPC` / `10.0.0.0/16` | Provides an isolated virtual network |
| Availability Zone 1 | `ap-south-1a` | Hosts one set of public/private subnets |
| Availability Zone 2 | `ap-south-1b` | Hosts the second set of public/private subnets |
| Public Subnet AZ1 | `10.0.1.0/24` | Public network segment |
| Private Subnet AZ1 | `10.0.2.0/24` | Private network segment |
| Public Subnet AZ2 | `10.0.3.0/24` | Public network segment in second AZ |
| Private Subnet AZ2 | `10.0.4.0/24` | Private network segment in second AZ |
| Internet Gateway | `Q2-Internet-Gateway` | Provides internet connectivity for public routing |
| NAT Gateway | `Q2-NAT-Gateway` | Provides outbound internet access for private subnets |
| Public Route Table | `Q2-Public-Route-Table` | Routes public traffic to the Internet Gateway |
| Private Route Table | `Q2-Private-Route-Table` | Routes private outbound traffic to the NAT Gateway |
| Security Group | `Q2-Web-Security-Group` | Controls inbound and outbound traffic |

---

## 4. VPC

**Name:** `Q2-Custom-VPC`  
**CIDR:** `10.0.0.0/16`

Amazon VPC provides the isolated virtual network in which the project subnets and networking components are configured.

---

## 5. Availability Zones

The architecture uses two Availability Zones:

- `ap-south-1a`
- `ap-south-1b`

The subnets are distributed across these two Availability Zones.

---

## 6. Subnets

### Public Subnets

**Q2-Public-Subnet-AZ1**
- AZ: `ap-south-1a`
- CIDR: `10.0.1.0/24`

**Q2-Public-Subnet-AZ2**
- AZ: `ap-south-1b`
- CIDR: `10.0.3.0/24`

The public subnets are associated with the public route table.

### Private Subnets

**Q2-Private-Subnet-AZ1**
- AZ: `ap-south-1a`
- CIDR: `10.0.2.0/24`

**Q2-Private-Subnet-AZ2**
- AZ: `ap-south-1b`
- CIDR: `10.0.4.0/24`

The private subnets are associated with the private route table.

---

## 7. Internet Gateway

**Name:** `Q2-Internet-Gateway`

The Internet Gateway is attached to `Q2-Custom-VPC`.

It provides a path between the VPC and the internet for traffic routed through the public route table.

Public default route:

```text
0.0.0.0/0 → Internet Gateway
```

---

## 8. NAT Gateway

**Name:** `Q2-NAT-Gateway`

The NAT Gateway is deployed in:

```text
Q2-Public-Subnet-AZ1
```

It provides outbound internet connectivity for resources in the private subnets.

Private default route:

```text
0.0.0.0/0 → NAT Gateway
```

---

## 9. Route Tables

### Public Route Table

**Name:** `Q2-Public-Route-Table`

Associated with:

- `Q2-Public-Subnet-AZ1`
- `Q2-Public-Subnet-AZ2`

Route:

```text
0.0.0.0/0 → Q2-Internet-Gateway
```

**Purpose:** Routes internet-bound traffic from public subnets through the Internet Gateway.

### Private Route Table

**Name:** `Q2-Private-Route-Table`

Associated with:

- `Q2-Private-Subnet-AZ1`
- `Q2-Private-Subnet-AZ2`

Route:

```text
0.0.0.0/0 → Q2-NAT-Gateway
```

**Purpose:** Routes outbound internet traffic from private subnets through the NAT Gateway.

---

## 10. Security Group

**Name:** `Q2-Web-Security-Group`

The Security Group acts as a virtual firewall for associated AWS resources.

### Inbound Rules

```text
HTTP | Port 80 | Source: 0.0.0.0/0
SSH  | Port 22 | Source: My IP
```

### Outbound Rules

```text
All traffic → 0.0.0.0/0
```

SSH access is restricted to the user's IP.

---

## 11. Network Traffic Flow

### Public Subnet

```text
Public Resource
      ↓
Public Route Table
      ↓
Internet Gateway
      ↓
Internet
```

### Private Subnet

```text
Private Resource
      ↓
Private Route Table
      ↓
NAT Gateway
      ↓
Internet Gateway
      ↓
Internet
```

---

## 12. Purpose of Each Networking Component

### VPC
Provides an isolated virtual network for AWS resources.

### Availability Zones
Provide separate locations within the AWS Region for distributing network resources.

### Public Subnet
Provides a subnet whose route table has a route to the Internet Gateway.

### Private Subnet
Provides a subnet without a direct route to the Internet Gateway.

### Internet Gateway
Provides internet connectivity for resources whose subnet routing directs traffic to it.

### NAT Gateway
Allows resources in private subnets to initiate outbound connections to the internet.

### Route Tables
Control where network traffic from associated subnets is sent.

### Security Group
Controls allowed inbound and outbound traffic for associated AWS resources.

---

## 13. Configuration Summary

```text
VPC:
Q2-Custom-VPC
10.0.0.0/16

Availability Zones:
ap-south-1a
ap-south-1b

Public Subnets:
Q2-Public-Subnet-AZ1  → 10.0.1.0/24
Q2-Public-Subnet-AZ2  → 10.0.3.0/24

Private Subnets:
Q2-Private-Subnet-AZ1 → 10.0.2.0/24
Q2-Private-Subnet-AZ2 → 10.0.4.0/24

Internet Gateway:
Q2-Internet-Gateway

NAT Gateway:
Q2-NAT-Gateway

Public Route Table:
Q2-Public-Route-Table

Private Route Table:
Q2-Private-Route-Table

Security Group:
Q2-Web-Security-Group
```

---

## 14. Conclusion

The custom VPC network architecture was configured using two Availability Zones, public and private subnets, an Internet Gateway, a NAT Gateway, route tables, and a Security Group.

The architecture demonstrates network segmentation, routing, internet connectivity, and traffic control using AWS networking components.
