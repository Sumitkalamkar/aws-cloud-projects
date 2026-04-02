# VPC with Public & Private Subnet + NAT Gateway 🌐

A production-ready AWS VPC setup with public and private subnets, Internet Gateway, and NAT Gateway — deployed fully via CloudFormation.

---

## Architecture

```
Internet
    ↓
Internet Gateway (gfg-internet-gateway)
    ↓
VPC (10.0.0.0/16)
    ├── Public Subnet (10.0.1.0/24)  — ap-south-1a
    │       ↓
    │   NAT Gateway (with Elastic IP)
    │       ↓
    └── Private Subnet (10.0.2.0/24) — ap-south-1b
            ↓
        Private Route Table → NAT Gateway → Internet
```

---

## AWS Services Used

| Service | Purpose |
|---|---|
| VPC | Isolated network with CIDR `10.0.0.0/16` |
| Public Subnet | Hosts internet-facing resources (e.g. ALB, Bastion) |
| Private Subnet | Hosts internal resources (e.g. App servers, DB) |
| Internet Gateway | Allows public subnet to reach the internet |
| NAT Gateway | Allows private subnet to reach internet (outbound only) |
| Elastic IP | Static public IP attached to NAT Gateway |
| Route Tables | Controls traffic routing for each subnet |

---

## Subnet Design

| Subnet | CIDR | AZ | Public IP | Use Case |
|---|---|---|---|---|
| Public | `10.0.1.0/24` | ap-south-1a | Yes (auto-assigned) | Load Balancers, Bastion Host |
| Private | `10.0.2.0/24` | ap-south-1b | No | App Servers, Databases |

---

## Routing Rules

### Public Subnet Route Table
| Destination | Target |
|---|---|
| `10.0.0.0/16` | Local (VPC internal) |
| `0.0.0.0/0` | Internet Gateway |

### Private Subnet Route Table
| Destination | Target |
|---|---|
| `10.0.0.0/16` | Local (VPC internal) |
| `0.0.0.0/0` | NAT Gateway |

---

## Deployment

### Steps

1. Open **AWS CloudFormation → Create Stack**
2. Upload `template.yaml`
3. Give the stack a name e.g. `vpc-nat-stack`
4. Leave all options as default
5. Click **Submit**

### Resources Created After Deployment

- `my-gfg-vpc` — VPC
- `public-subnet-gfg` — Public Subnet
- `private-subnet-gfg` — Private Subnet
- `gfg-internet-gateway` — Internet Gateway
- `public-rt-gfg` — Public Route Table
- `private-rt-gfg` — Private Route Table
- `gfg-eip` — Elastic IP
- `gw NAT` — NAT Gateway

---

## Key Design Decisions

- **MapPublicIpOnLaunch: true** on public subnet — EC2 instances launched here automatically get a public IP
- **NAT Gateway in Public Subnet** — NAT Gateway needs internet access itself to forward private subnet traffic
- **Elastic IP on NAT Gateway** — gives a fixed outbound IP for firewall whitelisting
- **DependsOn: AttachGateway** — ensures Internet Gateway is attached to VPC before routes are created
- **Private subnet has no direct internet route** — traffic only flows outbound through NAT, never inbound

---

## Why NAT Gateway?

| Scenario | Solution |
|---|---|
| EC2 in private subnet needs to download packages | NAT Gateway forwards request, returns response |
| External user tries to connect to private EC2 | Blocked — NAT only allows outbound |
| EC2 in public subnet needs internet | Internet Gateway directly |

---

## Cost Note

> ⚠️ NAT Gateway charges apply per hour + per GB of data processed.
> Delete the stack when not in use to avoid charges.

---

## Region

Deployed in: `ap-south-1` (Mumbai)
