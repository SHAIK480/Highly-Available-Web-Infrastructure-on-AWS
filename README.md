# Highly Available Web Infrastructure on AWS with Auto Scaling and CDN

## Overview

This project demonstrates the deployment of a **production-grade, highly available and scalable web infrastructure** on AWS. The architecture spans multiple Availability Zones and integrates core AWS services including load balancing, auto scaling, a CDN, shared file storage, a managed database, and DNS configuration.

---

## Architecture Diagram

```
                          User
                           |
                     [Route 53 DNS]
                   mybproject.duckdns.org
                           |
                  [CloudFront CDN]
             dhxsjd0yojfl7.cloudfront.net
                           |
              [Application Load Balancer]
                      Bproject-alb
                     /            \
           [EC2 - AZ1]          [EC2 - AZ2]
         web-server-1          web-server-2
         eu-north-1a           eu-north-1b
               |                    |
        [EFS Shared Storage - fs-027064200a0bed4cf]
                           |
               [RDS MySQL - projectdb]
                      eu-north-1a

       [S3 Bucket - bproject-s3bucket-mahi]
         (Static assets served via CloudFront)
```

---

## AWS Services Used

| Service | Purpose |
|---|---|
| Amazon VPC | Isolated network with custom CIDR |
| Amazon EC2 | Web server instances (NGINX) |
| Amazon EFS | Shared file storage across instances |
| Amazon RDS | Managed MySQL database |
| Amazon S3 | Static asset storage |
| Amazon CloudFront | CDN for global content delivery |
| Application Load Balancer | Distributes HTTP traffic across instances |
| Auto Scaling Group | Automatic instance management |
| Amazon Route 53 | DNS hosted zone configuration |
| AWS Certificate Manager | SSL/TLS certificate request and DNS validation configuration |
| DuckDNS | Free domain for testing |

---

## Infrastructure Details

### Region
`eu-north-1` (Stockholm)

### VPC
| Field | Value |
|---|---|
| VPC ID | vpc-0686700b8c9b73fac |
| Name | Bproject-vpc |
| IPv4 CIDR | 10.0.0.0/16 |
| DNS Hostnames | Enabled |

### Subnets
| Name | Subnet ID | CIDR | Availability Zone |
|---|---|---|---|
| Bproject-subnet-public1-eu-north-1a | subnet-0574b67d034ab1aa6 | 10.0.0.0/20 | eu-north-1a |
| Bproject-subnet-public2-eu-north-1b | subnet-0a892625e6c41ac5a | 10.0.16.0/20 | eu-north-1b |

### EC2 Instances
| Name | Instance ID | Type | AZ | Public IP |
|---|---|---|---|---|
| web-server-1 | i-09666078d1d7c2fc7 | t3.micro | eu-north-1a | 13.61.181.140 |
| web-server-2 | i-0016d6b112fbf7311 | t3.micro | eu-north-1b | 16.171.139.17 |

### EFS
| Field | Value |
|---|---|
| File System ID | fs-027064200a0bed4cf |
| Name | Bproject-efs |
| Performance Mode | General Purpose |
| Throughput Mode | Elastic |
| Availability | Regional (multi-AZ) |

### RDS Database
| Field | Value |
|---|---|
| DB Identifier | projectdb |
| Engine | MySQL Community |
| Instance Class | db.t3.micro |
| Region & AZ | eu-north-1a |
| Status | Available |

### Auto Scaling Group
| Field | Value |
|---|---|
| ASG Name | Bproject-asg |
| Launch Template | Bproject-temp (lt-06aac56f0f565c3b0) |
| AMI | ami-0d1856c2239281718 (Bproject-ami) |
| Instance Type | t3.micro |
| Desired Capacity | 2 |
| Min / Max | 2 / 4 |
| Status | At desired capacity |

### CloudFront Distribution
| Field | Value |
|---|---|
| Distribution ID | E1W50SJAZUBD5N |
| Domain | dhxsjd0yojfl7.cloudfront.net |
| Price Class | All Edge Locations |
| Default Root Object | index.html |
| HTTP Versions | HTTP/2, HTTP/1.1, HTTP/1.0 |

### Route 53
| Field | Value |
|---|---|
| Hosted Zone | mybproject.duckdns.org |
| Hosted Zone ID | Z029097633R8WPOFW1C2U |
| Type | Public Hosted Zone |

---

## Implementation Steps

### Step 1 — VPC Configuration
- Created custom VPC `Bproject-vpc` with CIDR `10.0.0.0/16`
- Created two public subnets across `eu-north-1a` and `eu-north-1b`
- Attached Internet Gateway `Bproject-igw` to the VPC
- Configured Route Table `Bproject-rtb-public` with routes:
  - `0.0.0.0/0` → Internet Gateway (for internet access)
  - `10.0.0.0/16` → local (for internal VPC traffic)

### Step 2 — EC2 Web Servers
- Launched EC2 instances (`t3.micro`, Amazon Linux 2023) in each public subnet
- Installed and configured NGINX web server
- Created custom `index.html` web page hosted on NGINX
- Mounted EFS shared storage on each instance

### Step 3 — EFS Shared Storage
- Created Regional EFS file system `Bproject-efs`
- Installed `amazon-efs-utils` on EC2 instances
- Mounted EFS on EC2 instances using the NFS mount command
- Stored shared website files (`index.html`) on the EFS mount point

### Step 4 — RDS Database
- Launched MySQL RDS instance `projectdb` (db.t3.micro) in `eu-north-1a`
- Created security group `rds-sg` allowing inbound MySQL (port 3306) only from EC2 security group
- Verified connectivity from EC2 instances using `mysql-client`

### Step 5 — S3 Static Storage
- Created S3 bucket `bproject-s3bucket-mahi`
- Uploaded static assets: `static.html` and `reddy's.jpg`
- Configured bucket permissions for CloudFront access

### Step 6 — Application Load Balancer
- Created ALB `Bproject-alb` spanning both public subnets
- Created Target Group `Bproject-tg` (HTTP:80, Instance type)
- Registered `web-server-1` and `web-server-2` as targets
- Verified both targets as **Healthy**

### Step 7 — AMI and Auto Scaling
- Created custom AMI `Bproject-ami` (ami-0d1856c2239281718) from configured EC2 instance
- Created Launch Template `Bproject-temp` using the custom AMI
- Configured Auto Scaling Group `Bproject-asg` (min: 2, desired: 2, max: 4)
- ASG automatically maintains instance count and registers new instances with ALB

### Step 8 — CloudFront CDN
- Created CloudFront distribution `Bproject-cf`
- Configured CloudFront distribution with Application Load Balancer as origin
- Set `index.html` as the default root object
- Enabled all edge locations for best performance
- Verified CloudFront to ALB routing and web accessibility

### Step 9 — DNS Configuration
- Registered `mybproject.duckdns.org` via DuckDNS
- Created Route 53 hosted zone for `mybproject.duckdns.org`
- Configured CNAME record pointing to CloudFront distribution
- Requested SSL certificate via AWS Certificate Manager and created DNS validation CNAME record
- Configured Route 53 hosted zone and DNS records for testing

### Step 10 — Verification
- Verified CloudFront to ALB routing and web accessibility
- Verified CloudFront → ALB → EC2 request routing
- Confirmed EFS shared storage consistency across both EC2 instances
- Tested RDS MySQL database connectivity from EC2
- Verified Auto Scaling Group maintained healthy instance count

---

## Architecture Flow

```
User Request
    │
    ▼
DNS records configured for testing via DuckDNS and Route 53
    │
    ▼
CloudFront CDN (Content Delivery)
    │
    ▼
Application Load Balancer (Traffic Distribution)
    │
   ┌┴────────────────────┐
   ▼                     ▼
EC2 (eu-north-1a)    EC2 (eu-north-1b)
web-server-1         web-server-2
   │                     │
   └─────────┬───────────┘
             ▼
       EFS (Shared Storage)
       /var/www/html/
             │
             ▼
       RDS MySQL Database
       (projectdb)

S3 Bucket ──► CloudFront (Static Assets)
```

---

## Challenges Faced

| Challenge | Description |
|---|---|
| CloudFront Origin Configuration | Troubleshooting origin path and protocol settings when connecting CloudFront to the ALB. Required testing to ensure requests forwarded correctly without path errors. |
| DNS Validation Limitations with DuckDNS | DuckDNS does not support full nameserver delegation to Route 53, which prevented complete HTTPS custom domain validation via ACM. DNS records were configured for testing only. |
| NGINX Default Root Object | NGINX server root had to be aligned with the EFS mount point (`/var/www/html/`) to ensure CloudFront and ALB served the correct `index.html`. |
| EFS Mount Consistency | After mounting EFS on both EC2 instances, `index.html` had to be recreated on the shared mount point to ensure both instances served identical content. |
| RDS Security Group Configuration | Ensuring RDS only allowed inbound MySQL traffic (port 3306) from the EC2 security group — not open to the internet — required careful inbound rule setup. |

---

## Key Features

- **High Availability** — Instances spread across two Availability Zones
- **Auto Scaling** — ASG automatically adjusts capacity (2–4 instances)
- **Load Balancing** — ALB evenly distributes incoming HTTP traffic
- **Shared Storage** — EFS ensures all EC2 instances serve identical content
- **CDN** — CloudFront configured for content delivery via global edge locations
- **Managed Database** — RDS MySQL with security group isolation
- **DNS** — Route 53 hosted zone and DNS records configured for testing


