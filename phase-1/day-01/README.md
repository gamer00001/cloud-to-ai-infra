# Day 01: Regions, AZs and VPC design

> **Phase 1:** AWS Cloud Architecture · **Time:** 75–90 min · **Level:** Beginner–Intermediate
> **Cost:** Free (design only, no AWS resources) · **Exam:** SAA-C03, Secure + Resilient domains

[← Phase 1](../README.md) · Next: Day 02, Routing, gateways and network security

## What you'll learn

- How Regions, Availability Zones and edge locations differ, and how to choose a Region
- What a VPC is and how to pick a CIDR block that won't cause trouble later
- How to split a VPC into **public, private (app) and isolated (db)** tiers across 2 AZs
- Subnet IP maths on AWS (5 reserved addresses per subnet)
- How route tables, NAT gateways and security groups work together

## The scenario

A Rails/Node SaaS app (ALB → app servers → Redis + PostgreSQL) is moving from a single server to AWS. It has to survive a data-centre failure, keep the database off the internet, and leave room for future EKS and GPU inference nodes.

## Core concepts

### 1. Cloud fundamentals
- **IaaS** (EC2, VPC): you manage the OS and everything above it. **PaaS** (RDS): you manage data and the app. **SaaS**: you just use it.
- **Shared responsibility:** AWS secures the cloud itself (hardware, facilities, hypervisor). You secure what you put in it: IAM, security groups, patching, data. 📖 [What is cloud computing?](https://aws.amazon.com/what-is-cloud-computing/)

### 2. Regions and AZs
- **Region:** a separate geographic area. Your data stays there unless you copy it elsewhere.
- **AZ:** one or more isolated data centres inside a Region.
  - **Multi-AZ** survives a data-centre failure.
  - **Multi-region** survives a Region outage.
- Choose a Region by **latency to users, compliance and data residency, service and GPU availability, and price**. 📖 [Global infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)

### 3. VPC
- A VPC lives in **one Region** and spans all of its AZs.
- The IPv4 CIDR can be anywhere from `/16` to `/28`. Avoid `10.0.0.0/16`, which everyone uses, so future peering or VPN connections don't overlap.
- The primary CIDR **can't be changed** later. You can only add secondary blocks. 📖 [What is Amazon VPC?](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)

### 4. Subnets
- **Each subnet lives in exactly one AZ.** For high availability, create one subnet per tier in each AZ.
- AWS reserves **5 IPs** per subnet, so a `/24` has **251 usable** addresses.
- A subnet is **public** only if its route table has `0.0.0.0/0 → Internet Gateway`. The name doesn't make it public. 📖 [Configure subnets](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)

| Subnet type | Default route | Typical use |
|---|---|---|
| Public | `0.0.0.0/0 → IGW` | ALB, NAT gateway |
| Private | `0.0.0.0/0 → NAT` (outbound only) | App servers |
| Isolated | local only | Databases, caches |

## Lab: design a multi-AZ VPC (paper only)

You won't create any AWS resources today. Create these files in `phase-1/day-01/`:

1. **`subnet-plan.md`**: plan 6 `/24` subnets in VPC `10.20.0.0/16`:

   | Tier | AZ a | AZ b |
   |---|---|---|
   | Public | 10.20.0.0/24 | 10.20.1.0/24 |
   | App | 10.20.10.0/24 | 10.20.11.0/24 |
   | DB | 10.20.20.0/24 | 10.20.21.0/24 |

   For each subnet, list the first and last usable address and the usable count. Leave gaps between tiers so there's room to grow.

2. **`routing-plan.md`**: one route table per tier:
   - `rt-public`: local + `0.0.0.0/0 → IGW`
   - `rt-app-a` / `rt-app-b`: local + `0.0.0.0/0 → NAT` in the **same AZ**
   - `rt-db`: local only

   Then add the security-group chain: `sg-alb` (443 from the internet) → `sg-app` (3000 from `sg-alb`) → `sg-db` (5432/6379 from `sg-app`).

3. **`architecture.md`**: your Region choice and reasons, the AZ strategy, what each tier does, failure modes and cost notes.

4. **`architecture-diagram.mmd`**: a Mermaid diagram of the ALB, app servers, Redis and PostgreSQL. GitHub renders Mermaid automatically.

**Cleanup:** none needed, since nothing was created.

### Done when
- [ ] 6 non-overlapping subnets across 2 AZs, with correct IP maths (251 usable per `/24`)
- [ ] The DB tier has no route to an IGW or NAT
- [ ] Security-group rules reference other SGs instead of CIDRs wherever possible
- [ ] The Region choice is justified
- [ ] The diagram renders on GitHub

## My solution

Try the lab yourself first, then compare:

| File | What it covers |
|---|---|
| [architecture.md](architecture.md) | Region choice, AZ strategy, tiers, failure modes, cost |
| [subnet-plan.md](subnet-plan.md) | 6 subnets with IP maths and room to grow |
| [routing-plan.md](routing-plan.md) | Route tables, NAT per AZ, SG chain |
| [architecture-diagram.mmd](architecture-diagram.mmd) | Mermaid diagram |

## Self-check

Answer these yourself first, then expand to check.

<details><summary>1. What makes a subnet public?</summary>

Its route table has a route `0.0.0.0/0 → Internet Gateway`. The name has nothing to do with it.
</details>

<details><summary>2. How many usable IPs are in an AWS /24 subnet?</summary>

251. That's 256 minus the 5 addresses AWS reserves: network, VPC router, DNS, one reserved for future use, and broadcast.
</details>

<details><summary>3. Why deploy one NAT gateway per AZ, and what does it cost you?</summary>

A NAT gateway lives in one AZ. If you only have one and its AZ fails, private subnets in the other AZs lose outbound internet access. One NAT per AZ, with per-AZ route tables, removes that single point of failure and avoids cross-AZ data charges. The cost is paying for each additional NAT (hourly plus per-GB). For labs and dev, a single NAT (or none) is a reasonable trade-off.
</details>

<details><summary>4. Why reference security groups instead of CIDR ranges?</summary>

It's least privilege: only resources that have the SG attached are allowed in, not everything in an IP range. It also keeps working as Auto Scaling replaces instances and as subnets change, and `sg-alb → sg-app → sg-db` reads like the architecture. Use CIDRs for sources that aren't your own SGs, such as the internet or a VPN range.
</details>

<details><summary>5. How do I decide the source and destination in SG rules?</summary>

Trace each arrow in the request flow, `A → B on port P`. B gets an **inbound** rule with source A, and A gets an **outbound** rule with destination B. In route tables, *destination* is where the packet is going and *target* is the next hop. When several routes match, the most specific one wins.
</details>

## Common mistakes

- Using `10.0.0.0/16` and later finding it overlaps with a partner's or the office VPN's network
- Calling a subnet "public" without adding the IGW route, or giving the DB subnet a route to the internet
- Putting all app servers in one AZ "for now"
- Opening SSH (22) to `0.0.0.0/0`. Use SSM Session Manager instead.
- Planning `/24` app subnets for EKS, where every pod takes a VPC IP. Plan larger blocks.

## References

- [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)
- [What is Amazon VPC?](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Configure subnets](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)