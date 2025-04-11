# High Availability Portfolio Project

---

## Overview

This project is an extension of project 1 with a few additions:
- Google oAuth Login
- Load balancing with two EC2 instances
- HAProxy to connect Route 53 with Instances

---

## Deployment Architecture

```
            collinfung.design (Route 53/DNS)
                      |
                  HAProxy (54.201.89.202)
                 /        \
         Amazon EC2 #1    Amazon EC2 #2
            with Nginx     with Nginx
       (44.245.174.10)    (54.202.126.21)


```

---

## OAuth Setup

Used Google Cloud Console to configure:

- Authorized JavaScript origins: http://collinfung.design
- Authorized redirect URIs: http://collinfung.design/auth/google/callback


1. **Create a new project on Google Cloud Console**  
   https://console.developers.google.com/  
   Click on “New Project”, give it a name (e.g., `portfolio-oauth`), and create it.

2. **Enable OAuth APIs**  
   Navigate to “APIs & Services” > “Library”  
   Search, add, and enable:
   - Google+ API

3. **Set up the OAuth**  
   - User type: External
   - Fill out details info to complete setup

4. **Create credentials**  
   - Go to “Credentials” > “Create Credentials” > “OAuth client ID”
   - Application type: Web Application
     
5. **Create env file**
   - Copy Client ID and Client Secret from additional information to .env file.

---

## Setup Servers
1. ***Set up one nginx server**
2. ***SCP files to nginx server**
4. **Install Nginx on server**
   - Install Node.js, Nginx, and clone the same Express app

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
5. **Clone a 2nd server from the intial setup**  
6. **Run the Express app**  
   Either using `node server.js` 

7. **Set up a 3rd server: HAProxy**  
   Install HAProxy:
```bash
sudo apt update
sudo apt install haproxy -y
```

Edit `/etc/haproxy/haproxy.cfg` and replace ipaddress with AWS IPv4

```haproxy
frontend http_front
    bind *:80
    stats uri /haproxy?stats
    default_backend aws_backends

backend nginx_servers
    balance roundrobin
    server aws1 'ipaddress':80 check
    server aws2 'ipaddress':80 check
```

## Explanation of HAProxy Configuration

- `frontend http_front`: Entry point for all traffic on port 80
- `bind *:80`: Listens on all IPs on port 80
- `default_backend aws_backends`: Defines the backend pool for routing
- `balance roundrobin`: Each server balances the load evenly in rotation
- `server aws# 'ipaddress':80 check`: Adds a backend server with health checking enabled

---


8. **Restart HAProxy**  
```bash
sudo systemctl restart haproxy
sudo systemctl enable haproxy
```

9. **DNS Configuration**  
   Point your Route53/domain 'A' record to the HAProxy **public IP** 


---




## Common Troubleshooting

**Issue #1 - Google OAuth redirect URI mismatch**
- Used https:// instead of https// in redirect URIs in Google Cloud Console
_Solution_: Always use http and ensure domain is not redirected to https directly


---

## Final Deliverables

- OAuth-secured portfolio
- GitHub contribution calendar embedded
- HAProxy load balancing


## Final Site
- http://collinfung.design
