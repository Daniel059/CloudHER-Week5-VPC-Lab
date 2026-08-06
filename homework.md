# Homework — Multi-Subnet VPC Design

**CloudHER by WIICA — Week 5**  
Companion file to the main [README.md](README.md) (Class Lab).

---

## Goal

Design a more realistic multi-subnet VPC. This prepares you for NAT Gateways, databases, load balancers, and multi-AZ architectures.

The idea is simple: a real network can have **public-facing resources** and **protected private areas** at the same time.

---

## Requirements

### 1. Create the VPC and Subnets

| Resource              | Value / Name        | Notes                              |
|-----------------------|---------------------|------------------------------------|
| VPC                   | `10.1.0.0/16`       | Name it clearly (e.g. `Homework-VPC`) |
| Public Subnet A       | `10.1.1.0/24`       | Enable auto-assign public IPv4     |
| Public Subnet B       | `10.1.2.0/24`       | Enable auto-assign public IPv4     |
| Private Subnet A      | `10.1.11.0/24`      | No public IP                       |
| Private Subnet B      | `10.1.12.0/24`      | No public IP                       |

> Tip: Place the two public subnets in different Availability Zones if possible (good practice for future multi-AZ work).

### 2. Internet Gateway & Routing

- Create and attach one Internet Gateway to the VPC
- Create a **Public Route Table**
- Add route: `0.0.0.0/0` → Internet Gateway
- Associate the Public Route Table **only** with the two public subnets
- Confirm the private subnets are **not** associated with the public route table and have no direct route to the IGW

### 3. Launch EC2 Web Server

1. Launch **one** EC2 instance in **Public Subnet A**
2. Assign a public IPv4 address
3. Security Group:
   - SSH (22) from **My IP**
   - HTTP (80) from **Anywhere-IPv4**
4. Install Apache and deploy a simple webpage (you can reuse the class lab page)

### 4. Verification

Confirm:
- Private subnets are **not** associated with the Public Route Table
- Private subnets do **not** have a direct route to the Internet Gateway
- Website is reachable via the public IP of the EC2 instance

**Screenshot:** Full subnet list + route table associations → `Screenshots/09-homework-architecture.png`

---

## Bonus Challenge

1. Launch one EC2 instance in **Private Subnet A** **without** a public IPv4 address
2. Create an **EC2 Instance Connect Endpoint** and connect to the instance using its private IP address

### Bonus Explanation

#### Why the private instance cannot be reached directly from the internet

A private subnet has **no route to an Internet Gateway**.  
Even if the instance somehow had a public IP (which it doesn’t), there is no path for traffic from the internet to enter that subnet.  
In addition, private instances typically do not get a public IPv4 address, so there is nothing for the internet to target.  
This is intentional isolation: databases, internal APIs, and application servers stay hidden from the public internet.

#### How the EC2 Instance Connect Endpoint provides secure access

An EC2 Instance Connect Endpoint acts as a controlled entry point into the VPC.  
Instead of exposing SSH (port 22) to the internet, you connect to the endpoint, and AWS brokers a secure connection to the instance’s **private IP**.  
Traffic never needs a public IP on the target instance, and you don’t have to open broad inbound rules.  
It gives you browser-based or CLI access while keeping the instance fully private.

#### Why a NAT Gateway provides outbound internet access but not inbound SSH

A NAT Gateway sits in a public subnet and allows instances in private subnets to **initiate** outbound connections (for example, to download packages or call external APIs).  
Return traffic for those connections is allowed, but **new inbound connections from the internet are not**.  
SSH from the internet is a new inbound connection, so a NAT Gateway will not accept it.  
NAT is for outbound convenience; it is not a way to open inbound access.

---

## Bonus Experiment

Temporarily remove the default route (`0.0.0.0/0 → IGW`) from the public route table and observe:

1. **Can you still SSH into the EC2 instance?**  
   Usually yes if you are already connected, or if you use a method that doesn’t rely on the public path. New connections from the internet will fail once the route is gone.

2. **Can you still open the website in a browser?**  
   No. The browser request from the internet has no path into the VPC without the `0.0.0.0/0 → IGW` route.

3. **Why?**  
   The route table is what tells the subnet how to reach (and be reached from) the internet. Removing `0.0.0.0/0 → IGW` removes that path. The Internet Gateway still exists, but nothing is pointing traffic to it.

Restore the route afterward so the website works again.

---

## Reflection Questions & Answers

### 1. Why do we need a VPC?

A VPC gives you a logically isolated network inside AWS that you fully control.  
You choose the IP range, create subnets, decide what can talk to the internet, and apply security boundaries.  
Without a VPC you would be forced to use the default network, which is less flexible and harder to secure for real applications.

### 2. What is the purpose of a subnet?

A subnet divides the VPC into smaller network segments.  
This lets you place resources in different “neighborhoods” — for example, public-facing web servers in one subnet and private databases in another — and control routing and security per segment.

### 3. Why does a Public Subnet need an Internet Gateway?

An Internet Gateway is the door between the VPC and the public internet.  
A subnet only becomes public when its route table sends internet-bound traffic (`0.0.0.0/0`) to that Internet Gateway.  
Without the IGW (and the matching route), even a subnet with public IPs cannot communicate with the internet.

### 4. Why doesn’t a Private Subnet have direct internet access?

A private subnet is deliberately **not** associated with a route table that points `0.0.0.0/0` to an Internet Gateway.  
As a result there is no path from the internet into that subnet, and instances there usually have no public IP.  
This isolation protects sensitive resources such as databases and internal services.

### 5. What route enables internet access?

The route:

```
Destination: 0.0.0.0/0
Target:      Internet Gateway (igw-xxxxxxxx)
```

This tells the subnet: “For any destination outside the VPC, send the traffic to the Internet Gateway.”

### 6. What happens if you remove the `0.0.0.0/0` route?

Internet connectivity for that subnet is lost.  
- New inbound connections from the internet (including the website) fail  
- Outbound connections to the internet also fail (unless a NAT Gateway is used in a more advanced design)  
The Internet Gateway may still be attached to the VPC, but without the route nothing uses it.

### 7. Why should SSH be restricted to My IP?

SSH gives full administrative access to the instance.  
Allowing it from `0.0.0.0/0` means anyone on the internet can attempt to log in.  
Restricting it to your own IP (or using Session Manager / Instance Connect Endpoint) greatly reduces the attack surface.

### 8. Which AWS networking components made the website accessible?

Several components had to work together:

1. **VPC** — the isolated network  
2. **Public Subnet** — where the EC2 was placed  
3. **Internet Gateway** — the door to the internet  
4. **Route Table** with `0.0.0.0/0 → IGW` associated with the public subnet  
5. **Public IP** on the EC2 instance  
6. **Security Group** allowing inbound HTTP (port 80)  
7. **Apache** running and serving the webpage on the instance  

If any one of these is missing or misconfigured, the website will not be reachable from the internet.

---

## Deliverables Checklist (Homework)

- [ ] VPC `10.1.0.0/16` created  
- [ ] 2 Public Subnets (`10.1.1.0/24`, `10.1.2.0/24`) with auto-assign public IP  
- [ ] 2 Private Subnets (`10.1.11.0/24`, `10.1.12.0/24`)  
- [ ] Internet Gateway attached  
- [ ] Public Route Table with `0.0.0.0/0 → IGW` associated only with public subnets  
- [ ] One EC2 web server in a public subnet, website reachable  
- [ ] Verified private subnets have no direct IGW route  
- [ ] Screenshot saved as `Screenshots/09-homework-architecture.png`  

### Bonus (Optional)

- [ ] Private EC2 launched without public IP  
- [ ] EC2 Instance Connect Endpoint created and used  
- [ ] Written explanation completed (see above)  

---

## Cleanup Reminder

When finished:

1. Terminate all EC2 instances  
2. Delete the Internet Gateway (detach first)  
3. Delete subnets  
4. Delete route tables  
5. Delete the VPC  
6. Remove unused security groups / key pairs if desired  

Always clean up to avoid unexpected charges.

---

**CloudHER by WIICA** • Week 5 Homework
