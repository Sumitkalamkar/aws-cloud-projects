# ALB Path-Based Routing with EC2 🌐

A hands-on AWS project demonstrating how an Application Load Balancer (ALB) routes incoming HTTP requests to different EC2 instances based on the URL path — without any code, purely using AWS console configuration.

---

## Architecture

```
Internet
     ↓
ALB (PathRoutingALB)
     ↓
/products  →  TG-Products  →  EC2 (Products-Server)
/users     →  TG-Users     →  EC2 (Users-Server)
/payments  →  TG-Payments  →  EC2 (Payments-Server)
```

---

## AWS Services Used

| Service | Purpose |
|---|---|
| EC2 (x3) | Serves different content per path |
| Application Load Balancer | Receives all traffic, routes by path |
| Target Groups (x3) | Links each path rule to the correct EC2 |
| Security Groups | Controls inbound HTTP traffic on port 80 |

---

## EC2 Instances Created

| Instance Name | Content Served | Path |
|---|---|---|
| `Products-Server` | `Welcome to PRODUCTS page!` | `/products/index.html` |
| `Users-Server` | `Welcome to USERS page!` | `/users/index.html` |
| `Payments-Server` | `Welcome to PAYMENTS page!` | `/payments/index.html` |

All instances use:
- AMI: Amazon Linux 2023
- Instance type: t2.micro
- Security Group: `MyWebServerGroup`

---

## Setup Steps

### Step 1 — Launch 3 EC2 Instances

Each instance uses Apache (`httpd`) installed via User Data script on launch.

**Products-Server User Data:**
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
mkdir -p /var/www/html/products
echo "<h1>Welcome to PRODUCTS page!</h1>" > /var/www/html/products/index.html
```

**Users-Server User Data:**
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
mkdir -p /var/www/html/users
echo "<h1>Welcome to USERS page!</h1>" > /var/www/html/users/index.html
```

**Payments-Server User Data:**
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
mkdir -p /var/www/html/payments
echo "<h1>Welcome to PAYMENTS page!</h1>" > /var/www/html/payments/index.html
```

---

### Step 2 — Create 3 Target Groups

| Target Group | Health Check Path | Registered Instance |
|---|---|---|
| `TG-Products` | `/products/index.html` | Products-Server |
| `TG-Users` | `/users/index.html` | Users-Server |
| `TG-Payments` | `/payments/index.html` | Payments-Server |

All target groups use Protocol: `HTTP`, Port: `80`

---

### Step 3 — Create Application Load Balancer

| Field | Value |
|---|---|
| Name | `PathRoutingALB` |
| Scheme | Internet-facing |
| Protocol | HTTP, Port 80 |
| Default Action | Forward to `TG-Products` |

---

### Step 4 — Add Path Routing Rules

| Rule Name | Priority | Path Pattern | Forward To |
|---|---|---|---|
| `Users-Rule` | 1 | `/users*` | `TG-Users` |
| `Payments-Rule` | 2 | `/payments*` | `TG-Payments` |
| *(default)* | — | all other traffic | `TG-Products` |

---

## Testing

Once ALB is **Active** and all target groups show **Healthy**, test using the ALB DNS name:

```
http://PathRoutingALB-2085029620.ap-south-1.elb.amazonaws.com/products/index.html
→ Welcome to PRODUCTS page!

http://PathRoutingALB-2085029620.ap-south-1.elb.amazonaws.com/users/index.html
→ Welcome to USERS page!

http://PathRoutingALB-2085029620.ap-south-1.elb.amazonaws.com/payments/index.html
→ Welcome to PAYMENTS page!
```

> ⚠️ Always use `http://` not `https://`

---

## Security Group Rules

Inbound rule required on `MyWebServerGroup`:

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | `0.0.0.0/0` |

---

## Key Concepts Learned

- ALB listener rules evaluate conditions in **priority order**
- Path pattern `/users*` matches `/users`, `/users/`, `/users/index.html` etc.
- Default rule acts as a **catch-all** for unmatched paths
- Target group health checks must pass before ALB sends traffic
- One ALB can route to **multiple services** saving cost vs one LB per service

---

## Region

Deployed in: `ap-south-1` (Mumbai)
