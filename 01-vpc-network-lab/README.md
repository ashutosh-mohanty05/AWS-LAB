# AWS VPC Bastion Host & Private EC2 Lab

## Objective
To design a secure AWS network architecture with public and private subnets and access a private EC2 instance through a Bastion host.

This lab simulates how production environments protect internal servers from direct internet access.

---

## Architecture

User → Internet → Internet Gateway → Bastion Host (Public Subnet) → Private EC2 (Private Subnet)

Private EC2 accesses the internet through a NAT Gateway.

---

## AWS Services Used

- VPC
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- EC2
- Security Groups

---

## Network Design

VPC CIDR:
10.0.0.0/16

---

Subnets:
Public Subnet: 10.0.1.0/24
Private Subnet: 10.0.2.0/24

---

## Infrastructure Components

### Bastion Host
- Located in Public Subnet
- Has Public IP
- Used as a jump server to access private instances

### Private EC2
- Located in Private Subnet
- No public IP
- Accessible only through Bastion Host

### NAT Gateway
Allows private instances to access the internet for updates and package installations.

---

## Security Group Configuration

### Bastion Security Group

Inbound:
SSH (22) → My IP     ##It was not working so i did "SSH (22) → 0.0.0.0/0"

Outbound:
All Traffic → 0.0.0.0/0


### Private EC2 Security Group

Inbound:
SSH (22) → Bastion Security Group

Outbound:
All Traffic → 0.0.0.0/0

---

## Route Table Configuration

### Public Subnet Route Table

10.0.0.0/16 → local
0.0.0.0/0 → Internet Gateway


### Private Subnet Route Table

10.0.0.0/16 → local
0.0.0.0/0 → NAT Gateway


---

## SSH Connection Flow

Step 1 — Connect to Bastion Host

ssh -i my-1st-instance.pem ec2-user@BASTION_PUBLIC_IP


Step 2 — Connect to Private EC2

ssh -i my-1st-instance.pem ec2-user@PRIVATE_IP


Example:

ssh -i my-1st-instance.pem ec2-user@10.0.2.220


---

## Verification Steps

Inside Private EC2:
Check hostname
hostname

Test internet access via NAT
ping 8.8.8.8

Install updates
sudo dnf update


---

## Troubleshooting Performed

### 1. SSH Connection Timeout
Problem:
Could not connect to instances via SSH.

Root Cause:
Security Group outbound traffic was restricted.

Fix:
Allowed outbound rule:

All Traffic → 0.0.0.0/0


---

### 2. Bastion Host Cannot Access Internet
Problem:
`ping google.com` was failing.

Root Cause:
Outbound traffic blocked in Security Group.

Fix:
Enabled outbound access to internet.

---

### 3. Private EC2 Cannot Reach Internet
Problem:
Private instance could not access external services.

Checks performed:
- Verified NAT Gateway status
- Verified private subnet route table
- Verified outbound security group rules

Fix:
Outbound rule enabled for private instance.

---

### 4. SSH Key Error
Error:

Load key "my-1st-instance.pem": error in libcrypto


Cause:
Key file corrupted or permission issue.

Fix:
Set correct permission:

chmod 400 my-1st-instance.pem


---

## Key Concepts Learned

- Public vs Private Subnet architecture
- Bastion Host pattern
- NAT Gateway usage
- Security Group inbound vs outbound rules
- Route table configuration
- SSH access to private instances
- AWS network troubleshooting

---

## Production Use Case

This architecture is commonly used to protect backend servers such as:

- Application servers
- Databases
- Internal services

Private resources remain inaccessible from the internet and are only accessed through controlled entry points.

---

## Cost Management

After completing the lab, the following resources were deleted to avoid charges:

- EC2 instances
- NAT Gateway
- Elastic IP
