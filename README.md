# AWS Cloud Projects ☁️

A collection of hands-on AWS projects covering real-world cloud architecture patterns — including serverless pipelines, networking, AI/ML integration, and load balancing.

> Built and deployed on AWS (ap-south-1 — Mumbai region)

---

## Projects

### 1. ALB Path-Based Routing 🌐

**Folder:** `alb-path-routing/`

Route HTTP traffic to different EC2 instances based on URL path using an Application Load Balancer.

```
Internet → ALB (PathRoutingALB)
             ├── /products → EC2 Products-Server
             ├── /users    → EC2 Users-Server
             └── /payments → EC2 Payments-Server
```

**Services:** EC2, ALB, Target Groups, Security Groups

---

### 2. AI-Ready Observability System 🔍

**Folder:** `ai-observability-system/`

Event-driven pipeline that automatically captures CloudWatch CPU alarms and stores structured alert data in DynamoDB — deployed via CloudFormation.

```
CloudWatch Alarm → SNS → SQS → Lambda → DynamoDB
```

**Services:** CloudWatch, SNS, SQS, Lambda (Python), DynamoDB, IAM, CloudFormation

---

### 3. S3 Image Object Detection with Rekognition 🖼️

**Folder:** `s3-rekognition-lambda/`

Serverless image analysis pipeline — automatically detects and labels objects in images when uploaded to S3 using Amazon Rekognition AI.

```
S3 Upload → Lambda Trigger → Rekognition → Detected Labels
```

**Services:** S3, Lambda (Python), Amazon Rekognition, IAM

---

### 4. VPC with Public & Private Subnet + NAT Gateway 🔒

**Folder:** `vpc-nat-gateway/`

Production-ready VPC setup with isolated public and private subnets, Internet Gateway for public access, and NAT Gateway for secure outbound-only internet access from private subnet — deployed via CloudFormation.

```
Internet → Internet Gateway → Public Subnet (10.0.1.0/24)
                                      ↓
                               NAT Gateway
                                      ↓
                            Private Subnet (10.0.2.0/24)
```

**Services:** VPC, Subnets, Internet Gateway, NAT Gateway, Elastic IP, Route Tables, CloudFormation

---

### 5. Docker - Complete Hands-On Guide 🐳

**Folder:** `docker-wordpress-mysql/`

A complete Docker guide covering image pull/push, container creation, Dockerfile keywords, Docker networking, and running a **WordPress + MySQL** multi-container application on a shared custom network.

```
Browser (port 8080)
       ↓
┌──────────────────┐        ┌─────────────────────┐
│  WordPress        │◄──────►│  MySQL               │
│  Container        │        │  Container           │
│  Port: 80         │        │  Port: 3306          │
└──────────────────┘        └─────────────────────┘
       └──────────── wp-network ────────────────────┘
```

**What's covered:**
- Pull and push Docker images
- Create and manage containers
- Write a Dockerfile and understand its keywords
- Create Docker networks
- Run WordPress + MySQL on the same Docker network

**Services/Tools:** Docker, Docker Hub, Docker Networking, Dockerfile, WordPress, MySQL

---

## Skills Demonstrated

| Skill | Projects |
|---|---|
| EC2 & Load Balancing | ALB Path Routing |
| Serverless Architecture | Observability System, Rekognition |
| Networking & VPC Design | VPC + NAT Gateway |
| Infrastructure as Code | Observability System, VPC (CloudFormation) |
| Event-Driven Architecture | Observability System, Rekognition |
| AI/ML Integration | Rekognition |
| Containerization | Docker WordPress + MySQL |
| IAM & Security | All Projects |

---

## Repo Structure

```
aws-cloud-projects/
│
├── alb-path-routing/
│   └── README.md
│
├── ai-observability-system/
│   ├── template.yaml
│   └── README.md
│
├── s3-rekognition-lambda/
│   ├── lambda_function.py
│   └── README.md
│
├── vpc-nat-gateway/
│   ├── template.yaml
│   └── README.md
│
└── docker-wordpress-mysql/
    ├── Dockerfile
    └── README.md
```

---

## Author

**Sumit Kalamkar**
AWS Cloud Enthusiast | Hands-on Builder
🔗 [GitHub](https://github.com/Sumitkalamkar)
