# AWS EC2 Load Balancing Lab (HAProxy + Nginx)

This project demonstrates a simple, highly-available web tier on AWS using:
- **2x EC2 web servers** running **Nginx** (`web-01`, `web-02`)
- **1x EC2 load balancer** running **HAProxy** (`lb-01`)
- **Round-robin load balancing** and **health-check based failover**

## Architecture

Client (Internet)
   |
   v
[ lb-01 (HAProxy, public) ]
   |
   +--> [ web-01 (Nginx) ]
   |
   +--> [ web-02 (Nginx) ]

## What this proves
- Linux service installation and management (systemd)
- Layer 7 HTTP load balancing (HAProxy)
- Health checks and backend failure handling
- Basic AWS networking concepts (public access to LB, controlled access to web tier)

## AWS Security Group Design (recommended)
### lb-01 Security Group (sg-lb)
Inbound:
- TCP 22 from my IP
- TCP 80 from 0.0.0.0/0

### web-01 & web-02 Security Group (sg-web)
Inbound:
- TCP 22 from my IP (or via bastion)
- TCP 80 from **sg-lb only**

## Validation / Proof
### Load balancing (round robin)
See: `evidence/curl-loadbalancing.txt`

Expected behavior:
- Requests to the LB alternate responses between web-01 and web-02.

### Failover test
See: `evidence/failover-test.txt`

Procedure:
- Stop Nginx on one backend node
- LB continues serving traffic via the healthy node

## Configuration Files
- HAProxy config: `haproxy/haproxy.cfg`
- Web server pages:
  - `nginx/web-01.index.html`
  - `nginx/web-02.index.html`

## Next steps
- Automate provisioning/configuration with **Ansible**
- Add HAProxy stats endpoint (restricted to my IP)
- Add Prometheus/Grafana monitoring for SRE-style visibility
