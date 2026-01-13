# AWS EC2 Load Balancing Lab (HAProxy + Nginx)

![Last Commit](https://img.shields.io/github/last-commit/danheck12/aws-haproxy-lab)
![Repo Size](https://img.shields.io/github/repo-size/danheck12/aws-haproxy-lab)
![Stars](https://img.shields.io/github/stars/danheck12/aws-haproxy-lab?style=social)

Hands-on AWS lab demonstrating **Layer 7 HTTP load balancing** using HAProxy and two Nginx backends on EC2.  
Designed to validate Linux service management, basic AWS networking design, and operational verification.

- **lb-01** – HAProxy (public entrypoint)
- **web-01**, **web-02** – Nginx backends
- Round-robin balancing + health-check based failover

---

## Table of Contents

- [Architecture](#architecture)
- [What This Proves](#what-this-proves)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Validation](#validation)
- [Security Group Design](#security-group-design)
- [Next Steps](#next-steps)

---

## Architecture

Client (Internet)
|
v
[ lb-01 (HAProxy, public) ]
|
|
v v
[ web-01 (Nginx) ] [ web-02 (Nginx) ]

yaml
Copy code

- HAProxy listens on port 80 and distributes traffic to backend Nginx servers.
- Backends are accessed via private IPs.
- Health checks ensure only healthy nodes receive traffic.

---

## What This Proves

- Linux package installation and service management
- HAProxy configuration: frontend, backend, health checks, failover
- Basic AWS networking design:
  - public access to load balancer
  - restricted access to backend tier
- Operational validation using real traffic and failure testing
- Evidence collection for reproducibility

---

## Repository Structure

.
├── haproxy/ # HAProxy configuration
│ └── haproxy.cfg
├── nginx/ # Backend content for each node
│ ├── web-01.index.html
│ └── web-02.index.html
├── evidence/ # Proof (curl output, failover tests)
│ ├── curl-loadbalancing.txt
│ └── failover-test.txt
└── diagrams/ # Architecture diagrams (optional)

yaml
Copy code

---

## Getting Started

### Prerequisites

- AWS account with EC2 + Security Group permissions
- SSH keypair for EC2 access
- 3 EC2 instances (Ubuntu 22.04+ recommended):
  - `lb-01`
  - `web-01`
  - `web-02`
- Basic familiarity with SSH and Linux package management

---

### 1) Create EC2 Instances

Launch three instances in the same VPC/subnet:

| Name   | Role     | Public IP | Private IP |
|--------|----------|-----------|------------|
| lb-01  | HAProxy  | Yes       | Yes        |
| web-01 | Nginx    | Optional  | Yes        |
| web-02 | Nginx    | Optional  | Yes        |

Tag them accordingly for clarity.

---

### 2) Install and Configure Nginx on Backends

On **web-01** and **web-02**:

```bash
sudo apt-get update
sudo apt-get install -y nginx
sudo systemctl enable --now nginx
Replace the default page with the per-node file from this repo.

For web-01:

bash
Copy code
sudo tee /var/www/html/index.html >/dev/null <<'EOF'
<contents of nginx/web-01.index.html>
EOF
For web-02:

bash
Copy code
sudo tee /var/www/html/index.html >/dev/null <<'EOF'
<contents of nginx/web-02.index.html>
EOF
Validate:

bash
Copy code
curl -s localhost | head
3) Install and Configure HAProxy
On lb-01:

bash
Copy code
sudo apt-get update
sudo apt-get install -y haproxy
sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
Copy the repo config into place:

bash
Copy code
sudo nano /etc/haproxy/haproxy.cfg
Update the backend section with private IPs of web-01 and web-02.

Validate config:

bash
Copy code
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
Restart service:

bash
Copy code
sudo systemctl restart haproxy
sudo systemctl status haproxy --no-pager
Configuration
HAProxy config: haproxy/haproxy.cfg

Backend pages:

nginx/web-01.index.html

nginx/web-02.index.html

Ensure backend IPs in HAProxy match your actual EC2 private IPs.

Validation
1) Load Balancing (Round Robin)
From your local machine:

bash
Copy code
LB_PUBLIC_IP="<lb-01 public ip>"

for i in {1..6}; do
  curl -s http://$LB_PUBLIC_IP | head -n 2
  echo "----"
done
Expected:

Output alternates between web-01 and web-02

2) Failover Test
On one backend (e.g. web-01):

bash
Copy code
sudo systemctl stop nginx
From your machine:

bash
Copy code
for i in {1..10}; do
  curl -s http://$LB_PUBLIC_IP | head -n 2
  echo "----"
done
Expected:

Traffic continues via the healthy backend only

Restart service:

bash
Copy code
sudo systemctl start nginx
Security Group Design
lb-01 Security Group (sg-lb)
Inbound:

TCP 22 from your IP

TCP 80 from 0.0.0.0/0

Outbound:

All (default)

web-01 / web-02 Security Group (sg-web)
Inbound:

TCP 22 from your IP (or via SSM)

TCP 80 from sg-lb only

Outbound:

All (default)
