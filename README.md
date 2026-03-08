# three-tier-architecture-design
# 3-Tier Highly Available Web Application on AWS

A production-ready **3-tier architecture** deployed in a single VPC across two Availability Zones (us-east-1a and us-east-1b) using AWS best practices for high availability, scalability, and security.

This design separates concerns into:
- **Presentation tier** (public-facing load balancer)
- **Application tier** (web/app servers in private subnets)
- **Data tier** (relational database with high availability)

![3-Tier AWS Architecture](3-tier-vpc-diagram.drawio.svg)  
*(Vector SVG – zoomable on GitHub. Edit in diagrams.net/draw.io by opening this file from the repo via File → Open → GitHub.)*

## Architecture Overview

- **Region**: us-east-1
- **VPC**: 10.0.0.0/16 – single VPC spanning multiple AZs
- **Availability Zones**: us-east-1a and us-east-1b

### Public Tier (Internet-facing)
- **Public Subnets**:
  - Public Subnet 1 – 10.0.1.0/24 (us-east-1a)
  - Public Subnet 2 – 10.0.2.0/24 (us-east-1b)
- **Internet Gateway** (IGW) attached to VPC
- **NAT Gateways** (one per AZ) placed in public subnets → enables outbound internet from private subnets
- **Public Route Table**: 0.0.0.0/0 → Internet Gateway

### Application Tier (Private)
- **Private Subnets**:
  - Private Subnet 1 – 10.0.3.0/24 (us-east-1a)
  - Private Subnet 2 – 10.0.3.0/24 (us-east-1b)  *(Note: same CIDR shown in diagram – in real setups use non-overlapping ranges like 10.0.4.0/24)*
- **Auto Scaling Group (ASG)** running EC2 instances ("servers") across both private subnets
- **Security Group** attached to instances – allows inbound only from ALB (least privilege)
- **Private Route Table**: 0.0.0.0/0 → NAT Gateway (outbound internet access)

### Data Tier (Private)
- **Amazon RDS (Multi-AZ)**:
  - **Primary database instance** in us-east-1a
  - **Standby database instance** (synchronous replica) in us-east-1b
  - Automatic failover (< 2 minutes typical, near-zero data loss)
- **DB Subnet Group**: Spans both private subnets
- **Security Group** for RDS – allows inbound only from application tier security group

### Key Traffic Flows
- **Inbound**: Internet → Application Load Balancer (ALB in public subnets) → Auto Scaling Group (private subnets) → RDS (private)
- **Outbound**: Private instances → NAT Gateway (public subnets) → Internet Gateway → external services

## Design Goals & Benefits

- **High Availability** — Multi-AZ everywhere (ALB, ASG, NAT, RDS standby)
- **Security** — No public IPs on application or database instances
- **Scalability** — Horizontal scaling via Auto Scaling Group
- **Cost-aware outbound** — NAT Gateways provide controlled internet egress
- **Isolation** — Proper 3-tier separation (public ↔ private app ↔ private DB)

## Production Recommendations & Next Steps

To take this from good learning setup to fully production-grade:

- Add **Application Load Balancer (ALB)** in public subnets (not shown yet) with:
  - HTTPS listener + free ACM certificate
  - HTTP → HTTPS redirect
  - Target groups pointing to the private ASG
- Attach **AWS WAF** to the ALB for OWASP protection
- Enable **CloudWatch** detailed monitoring, alarms, and auto-scaling policies (e.g., target tracking on CPU 50%)
- Add **RDS Proxy** in front of the database for connection pooling
- Use **IAM roles** for EC2 instances (no hard-coded credentials)
- Offload static assets to **S3 + CloudFront**
- Add **Route 53** alias record pointing to the ALB
- Consider **AWS Backup** or automated snapshots for RDS
- Deploy via **Infrastructure as Code** (Terraform, CloudFormation, or AWS CDK)

## How to View / Edit the Diagram

1. Click the SVG preview above (or open the file in this repo).
2. To edit: https://app.diagrams.net → **File → Open → GitHub** → select this repository and file.
   - Fully editable thanks to embedded diagram data.

Built with diagrams.net (draw.io) + AWS Architecture Icons.  
Feel free to fork, suggest improvements, or use as reference!

Questions? Open an issue or PR.
