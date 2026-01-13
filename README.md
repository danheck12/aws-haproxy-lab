 AWS EC2 Load Balancing Lab (HAProxy + Nginx)

![Last Commit](https://img.shields.io/github/last-commit/danheck12/aws-haproxy-lab)
![Repo Size](https://img.shields.io/github/repo-size/danheck12/aws-haproxy-lab)
![Stars](https://img.shields.io/github/stars/danheck12/aws-haproxy-lab?style=social)

A small AWS lab demonstrating **Layer 7 HTTP load balancing** with HAProxy and two Nginx backends on EC2.

- `lb-01`: HAProxy (public entrypoint)
- `web-01`, `web-02`: Nginx backends
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

---

## What This Proves
- Linux service installation and management (systemd)
- HAProxy configuration: frontend/backend, health checks, failover
- Basic AWS networking design:
  - public access to LB
  - controlled access to backend tier
- Operational validation with recorded evidence

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

pgsql
Copy code

---

## Getting Started

### Prerequisites
- AWS account access to create EC2 instances and security groups
- SSH keypair for EC2 access
- 3 Ubuntu instances recommended (or Amazon Linux 2 if you prefer):
  - `lb-01`, `web-01`, `web-02`

### 1) Create Security Groups
See [Security Group Design](#security-group-design) below.

### 2) Install and configure Nginx on web nodes
On **web-01** and **web-02**:

```bash
sudo apt-get update
sudo apt-get install -y nginx

# Replace the default index.html with the per-node file from this repo
# (copy web-01.index.html to web-01, web-02.index.html to web-02)
sudo tee /var/www/html/index.html >/dev/null <<'EOF'
REPLACE_ME
EOF

sudo systemctl enable --now nginx
curl -s localhost | head
Tip: the simplest way is to copy/paste the contents of:

nginx/web-01.index.html onto web-01

nginx/web-02.index.html onto web-02

3) Install and configure HAProxy on lb-01
On lb-01:

bash
Copy code
sudo apt-get update
sudo apt-get install -y haproxy
sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
Copy your repo config (haproxy/haproxy.cfg) into place and update backend IPs:

bash
Copy code
sudo nano /etc/haproxy/haproxy.cfg
# set web-01 and web-02 private IPs in the backend section
Then:

bash
Copy code
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
sudo systemctl status haproxy --no-pager
Configuration
HAProxy config: haproxy/haproxy.cfg

Backend pages:

nginx/web-01.index.html

nginx/web-02.index.html

Validation
Load balancing (round robin)
See: evidence/curl-loadbalancing.txt

From your machine:

bash
Copy code
LB_PUBLIC_IP="<lb-01 public ip>"
for i in {1..6}; do curl -s http://$LB_PUBLIC_IP | head -n 2; echo "----"; done
Expected:

responses alternate between web-01 and web-02

Failover test
See: evidence/failover-test.txt

Procedure:

Stop nginx on one backend:

bash
Copy code
sudo systemctl stop nginx
Confirm traffic still serves via the healthy node:

bash
Copy code
for i in {1..10}; do curl -s http://$LB_PUBLIC_IP | head -n 2; echo "----"; done
Start nginx again:

bash
Copy code
sudo systemctl start nginx
Security Group Design
lb-01 Security Group (sg-lb)
Inbound:

TCP 22 from your IP

TCP 80 from 0.0.0.0/0

web-01 & web-02 Security Group (sg-web)
Inbound:

TCP 22 from your IP (or via bastion/SSM)

TCP 80 from sg-lb only (LB → web tier)
