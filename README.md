# AWS Highly Available Web Architecture

A hands-on AWS infrastructure project demonstrating a highly available web application architecture using Amazon VPC, multi-AZ networking, NAT Gateways, an S3 Gateway Endpoint, EC2 Auto Scaling, an Application Load Balancer, a Target Group, a Bastion Host, and Apache2.

## Architecture

The project uses a VPC across **2 Availability Zones** with **2 public subnets** and **2 private subnets**. NAT Gateways are configured per AZ, and an S3 Gateway Endpoint provides private S3 connectivity.

```text
Internet
   |
   v
Internet-facing Application Load Balancer :80
   |
   v
Target Group + Health Checks
   |
   +---------------------------+
   |                           |
Private Subnet / AZ-1     Private Subnet / AZ-2
   |                           |
EC2 + Apache2              EC2 + Apache2
   ^                           ^
   |                           |
        EC2 Auto Scaling Group
             |
        Launch Template

Private subnet outbound traffic
        |             |
   NAT Gateway    NAT Gateway
        |             |
   Public subnet / AZ-1 and AZ-2
             |
       Internet Gateway

VPC -> S3 Gateway Endpoint -> S3

Administration:
Administrator -> Bastion Host -> SSH -> Private EC2
```

## AWS Services

| Service | Purpose |
|---|---|
| VPC | Isolated AWS network |
| Availability Zones | High availability across 2 AZs |
| Public Subnets | ALB, NAT and Bastion infrastructure |
| Private Subnets | Application/web EC2 instances |
| Internet Gateway | Public subnet internet connectivity |
| NAT Gateway | Outbound internet access for private instances |
| S3 Gateway Endpoint | Private S3 connectivity |
| EC2 Launch Template | Reusable instance configuration |
| Auto Scaling Group | Maintains and scales EC2 capacity |
| Application Load Balancer | Internet-facing HTTP entry point |
| Target Group | Routes traffic and performs health checks |
| Bastion Host | Administrative access to private instances |
| Apache2 | Web server |

## Network Configuration

The documented VPC uses:

```text
VPC CIDR: 10.0.0.0/16
Availability Zones: 2
Public subnets: 2
Private subnets: 2
NAT Gateways: 1 per AZ
S3 Endpoint: Gateway
```

The source screenshots also show the AWS VPC resource map connecting subnets, route tables and network connections.

## Launch Template

The documented Launch Template uses:

- Ubuntu 24.04 LTS
- `t3.micro`
- Key pair: `Arch_proj_keypair`
- VPC/security-group networking

The Launch Template is consumed by the Auto Scaling Group.

## Auto Scaling Group

The documented configuration includes:

- Desired capacity: **2**
- Minimum capacity: **2**
- Maximum capacity: **5**
- Target tracking scaling policy
- Average CPU utilization target: **70%**
- EC2 health checking

This provides a baseline of two application instances and allows the group to scale according to load.

## Bastion Host

A Bastion Host is created in a public subnet.

The documented administration flow is:

1. Create the Bastion Host.
2. Copy the PEM key to the Bastion Host using `scp`.
3. SSH from the Bastion Host to a private EC2 instance.
4. Install Apache2.
5. Verify the Apache service is running.

## Target Group

The Target Group uses:

```text
Protocol: HTTP
Port: 80
Target type: Instances
```

Health checks are configured so the Load Balancer can route traffic to healthy instances.

## Application Load Balancer

The project creates an **internet-facing Application Load Balancer** across the public subnets.

The documented flow is:

```text
Client
  |
  | HTTP :80
  v
Application Load Balancer
  |
  v
Target Group
  |
  +---- EC2 / Apache2
  |
  +---- EC2 / Apache2
```

## Validation

After creating the Load Balancer, its DNS name is opened in a browser. The final screenshot shows the **Apache2 Ubuntu default page**, confirming that the request reaches the web server through the Load Balancer.

## Implementation Sequence

1. Create the VPC.
2. Configure two Availability Zones.
3. Create two public and two private subnets.
4. Configure NAT Gateways.
5. Configure the S3 Gateway Endpoint.
6. Create the EC2 Launch Template.
7. Create the Auto Scaling Group.
8. Configure scaling and health checks.
9. Create the Bastion Host.
10. Copy the PEM key using `scp`.
11. SSH to a private EC2 instance.
12. Install Apache2.
13. Create the Target Group.
14. Register/validate EC2 targets.
15. Create the Application Load Balancer.
16. Configure the HTTP listener and Target Group.
17. Test using the ALB DNS name.

## Security / Production Hardening

The source project is a hands-on lab. For production, improve it by:

- Restricting SSH to a trusted CIDR or using AWS Systems Manager Session Manager.
- Avoiding `0.0.0.0/0` for administrative SSH.
- Allowing application EC2 traffic from the ALB security group rather than broad CIDRs.
- Using HTTPS with AWS Certificate Manager.
- Using IAM roles with least privilege.
- Enabling CloudTrail, CloudWatch and VPC Flow Logs.
- Managing secrets through Secrets Manager or Parameter Store.
- Using Infrastructure as Code and CI/CD for repeatable changes.

## Skills Demonstrated

- AWS VPC and CIDR planning
- Public/private subnet architecture
- Multi-AZ design
- Internet Gateway
- NAT Gateway
- VPC Endpoint
- EC2
- Launch Templates
- Auto Scaling Groups
- Target Tracking Scaling
- Application Load Balancer
- Target Groups and health checks
- Bastion Host administration
- SSH/SCP
- Linux administration
- Apache2
- AWS architecture troubleshooting

## Interview Explanation

> I designed and implemented a highly available AWS web architecture across two Availability Zones. I created a VPC with separate public and private subnets, configured NAT Gateways for private-subnet outbound connectivity, and added an S3 Gateway Endpoint. The application layer uses an EC2 Launch Template and Auto Scaling Group with a desired capacity of two and target tracking based on CPU utilization. An internet-facing Application Load Balancer distributes HTTP traffic to the EC2 instances through a Target Group with health checks. For administration, I used a Bastion Host to access the private instance and installed Apache2. Finally, I validated the architecture using the ALB DNS name and confirmed the Apache2 default page was accessible.

## Recommended GitHub Structure

```text
aws-high-availability-web-architecture/
├── README.md
├── docs/
│   ├── AWS_Architecture_Project_Guide.pdf
│   └── screenshots/
└── terraform/
    └── README.md
```

If you later add Terraform, keep networking, security, compute and load-balancing resources modular rather than placing the complete architecture in one `main.tf`.

## Future Improvements

- Terraform modules
- Remote Terraform state
- CI/CD pipeline
- HTTPS + ACM
- Route 53
- AWS WAF
- CloudWatch alarms
- SSM Session Manager
- IAM least-privilege roles
- Secrets Manager / Parameter Store
- Separate dev/stage/prod environments
- Blue/green or rolling deployments
