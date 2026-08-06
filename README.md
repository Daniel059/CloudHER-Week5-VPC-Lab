# CloudHER by WIICA — Week 5: Amazon VPC Networking Lab

![AWS](https://img.shields.io/badge/AWS-VPC-orange?logo=amazon-aws)
![Level](https://img.shields.io/badge/Level-Beginner%20to%20Intermediate-blue)
![Status](https://img.shields.io/badge/Status-Class%20Lab%20Completed-success)

Hands-on lab for building a custom Amazon VPC with public and private subnets, route tables, an Internet Gateway, and an EC2 web server.

**Course:** CloudHER by WIICA  
**Week:** 5 — Amazon VPC Networking

---

## Project Overview

In this project I designed and built a custom isolated network in AWS (a VPC), divided it into public and private subnets, connected it to the internet securely, and launched a web server reachable from the internet.

### Learning Objectives
- Understand VPC, subnet, CIDR, Internet Gateway, and Route Table
- Build a custom VPC from scratch (not the default VPC)
- Create public and private subnets
- Make a subnet public using an Internet Gateway + route table
- Launch and secure an EC2 instance inside a VPC
- Serve a simple website and troubleshoot connectivity end-to-end

---

## Architecture

### Class Lab Architecture

```
Internet
   │
Internet Gateway (CloudHER-IGW)
   │
┌──────────────────────────────────────────────┐
│              CloudHER-VPC (10.0.0.0/16)       │
│                                              │
│  ┌────────────────────┐  ┌────────────────┐  │
│  │  Public Subnet     │  │ Private Subnet │  │
│  │  10.0.1.0/24       │  │ 10.0.2.0/24    │  │
│  │                    │  │                │  │
│  │  ┌──────────────┐  │  │  (No public    │  │
│  │  │ EC2 + Apache │  │  │   resources)   │  │
│  │  │ Web Server   │  │  │                │  │
│  │  └──────────────┘  │  │                │  │
│  └────────────────────┘  └────────────────┘  │
└──────────────────────────────────────────────┘
```

### Homework Architecture

```
VPC: 10.1.0.0/16
├── Public Subnet A   10.1.1.0/24   ← EC2 Web Server
├── Public Subnet B   10.1.2.0/24
├── Private Subnet A  10.1.11.0/24
└── Private Subnet B  10.1.12.0/24
```

---

## Repository Structure

```
CloudHER-Week5-VPC-Lab/
├── README.md          ← This file
├── LICENSE
├── .gitignore
├── index.html         ← Webpage deployed on the EC2 instance
└── Screenshots/       ← Lab evidence screenshots
```

---

## Class Lab Steps

### 1. Create VPC
- Name: `CloudHER-VPC`
- CIDR: `10.0.0.0/16`

### 2. Create Subnets

**Public Subnet**
- Name: `Public-Subnet`
- CIDR: `10.0.1.0/24`
- Auto-assign public IPv4: Enabled

**Private Subnet**
- Name: `Private-Subnet`
- CIDR: `10.0.2.0/24`

### 3. Create & Attach Internet Gateway
- Name: `CloudHER-IGW`
- Attached to `CloudHER-VPC`

### 4. Create Public Route Table
- Name: `Public-RT`
- Route: `0.0.0.0/0` → Internet Gateway (`CloudHER-IGW`)
- Associated with **Public-Subnet only**

### 5. Launch EC2
- Name: `CloudHER-Web-Server`
- AMI: Amazon Linux 2023
- Instance type: `t3.micro`
- Subnet: Public-Subnet
- Auto-assign public IP: Enabled
- Security Group (`CloudHER-Web-SG`):
  - SSH (22) → My IP / Anywhere (for Instance Connect)
  - HTTP (80) → Anywhere-IPv4

### 6. Install Apache & Deploy Website

```bash
sudo dnf install httpd -y
sudo systemctl enable httpd
sudo systemctl start httpd
```

Webpage content is in `index.html` (copied to `/var/www/html/index.html`).

### 7. Verify
- `curl http://localhost` works inside the instance
- Website reachable at `http://<Public-IP>`

---

## Homework

| Resource            | Value              |
|---------------------|--------------------|
| VPC                 | `10.1.0.0/16`      |
| Public Subnet A     | `10.1.1.0/24`      |
| Public Subnet B     | `10.1.2.0/24`      |
| Private Subnet A    | `10.1.11.0/24`     |
| Private Subnet B    | `10.1.12.0/24`     |

- Attach Internet Gateway
- Public route table associated **only** with the public subnets
- Launch one EC2 web server in a public subnet
- Verify private subnets have no direct route to the Internet Gateway

### Bonus Challenge
- Launch an EC2 instance in a private subnet (no public IP)
- Create an EC2 Instance Connect Endpoint and connect using the private IP
- Explain:
  - Why the private instance cannot be reached directly from the internet
  - How the EC2 Instance Connect Endpoint provides secure access
  - Why a NAT Gateway provides outbound internet access but not inbound SSH

---

## Screenshots

| File | Description |
|------|-------------|
| `01-vpc.png` | VPC created with CIDR `10.0.0.0/16` |
| `02-subnets.png` | Public + Private subnets |
| `03-igw.png` | Internet Gateway attached |
| `04-route-table.png` | Route table with `0.0.0.0/0 → IGW` |
| `05-rt-association.png` | Route table associated only with Public Subnet |
| `06-ec2.png` | EC2 instance Running with Public IP |
| `07-security-group.png` | Security Group inbound rules |
| `08-website.png` | Browser showing the live website |
| `09-homework-architecture.png` | Homework multi-subnet layout |
| `10-bonus-private-access.png` | Bonus private instance access (optional) |

---

## Deliverables Checklist

### Class Lab
- [x] Custom VPC created (`CloudHER-VPC` / `10.0.0.0/16`)
- [x] Public Subnet + Private Subnet
- [x] Internet Gateway attached
- [x] Public Route Table with correct route and association
- [x] EC2 running in Public Subnet
- [x] Apache installed and website reachable from the internet
- [x] Screenshots saved in `Screenshots/`

### Homework
- [ ] VPC `10.1.0.0/16` with 2 public + 2 private subnets
- [ ] IGW + Public Route Table only on public subnets
- [ ] One EC2 web server in a public subnet
- [ ] Private subnets have no direct IGW route

### Bonus
- [ ] Private EC2 + EC2 Instance Connect Endpoint
- [ ] Written explanation of private access

---

## Troubleshooting Checklist

When the website does not open, check in this order:

1. EC2 instance is **Running**
2. Status checks passed
3. Public IPv4 is assigned
4. Security Group allows HTTP 80 from Anywhere
5. Apache is running (`sudo systemctl status httpd`)
6. `curl localhost` works inside the instance
7. Subnet route table has `0.0.0.0/0 → IGW`
8. Using `http://` (not `https://`)

---

## Reflection Questions

1. Why do we need a VPC?
2. What is the purpose of a subnet?
3. Why does a Public Subnet need an Internet Gateway?
4. Why doesn’t a Private Subnet have direct internet access?
5. What route enables internet access?
6. What happens if you remove the `0.0.0.0/0` route?
7. Why should SSH be restricted to My IP?
8. Which AWS networking components made the website accessible?

---

## Cleanup

When finished with the lab:

1. Terminate EC2 instance(s)
2. Detach and delete the Internet Gateway
3. Delete subnets
4. Delete route table(s)
5. Delete the VPC
6. Delete unused security groups and key pairs if desired

> Always clean up resources to avoid unexpected AWS charges.

---

## Technologies Used

- Amazon VPC
- Subnets (Public & Private)
- Internet Gateway
- Route Tables
- EC2 (Amazon Linux 2023)
- Security Groups
- Apache HTTP Server

---

**CloudHER by WIICA** • Week 5 VPC Lab
