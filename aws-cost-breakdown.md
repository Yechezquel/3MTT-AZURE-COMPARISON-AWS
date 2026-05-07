# AWS Cost Breakdown — Small Web Application

## Application Specification

A Linux-based web application serving a small e-commerce storefront.
The application runs on a single virtual machine with managed object storage for static assets.

### Compute
- **vCPU:** 2 cores
- **RAM:** 4 GB
- **Operating System:** Ubuntu 22.04 LTS (Linux)
- **Uptime:** 730 hours/month (24/7)

### Storage
- **OS/App disk:** 30 GB SSD (boot volume + application code)
- **Object storage:** 50 GB (product images, CSS/JS assets, log archives)
- **Monthly API requests:** 100,000 GET + 10,000 PUT

### Network
- **Monthly egress:** 100 GB (page loads, image serving to end users)
- **Ingress:** Free on AWS
- **Architecture:** Single AZ — Pattern B (single VM + object storage, no load balancer)

### Region
- **AWS Region:** us-east-1 (N. Virginia)
- **Rationale:** Lowest-cost AWS reference region. For a Nigeria-based production
  deployment, eu-west-1 (Ireland) would offer the best cost-to-latency trade-off
  from Lagos at approximately 30% lower cost than af-south-1 (Cape Town).

---

## AWS Services Used

| AWS Service | Purpose | Equivalent Need |
|---|---|---|
| EC2 t3.medium | Virtual machine | 2 vCPU, 4 GB RAM compute |
| EBS gp3 (30 GB) | Block storage | OS disk and application code |
| Amazon S3 Standard | Object storage | Static assets and backups |
| Data Transfer Out | Networking | Internet egress to end users |

---

## Monthly Cost Breakdown (On-Demand)

| Line Item | Configuration | Monthly Cost |
|---|---|---|
| EC2 t3.medium | Linux, 730 hrs, on-demand, us-east-1 | $30.37 |
| EBS gp3 | 30 GB, 3000 IOPS (free), 125 MBps (free) | $2.40 |
| Data transfer out | 100 GB to internet | $9.00 |
| Amazon S3 Standard | 50 GB storage | $1.15 |
| S3 requests | 100,000 GET + 10,000 PUT | $0.09 |
| **Total (On-Demand)** | | **$43.01/month** |

> **AWS Pricing Calculator estimate:**
> https://calculator.aws/#/estimate?id=ecb1d72711d2b3eae96c16334a9262098f606673

---

## EC2 Pricing Breakdown (from calculator)

The AWS calculator confirmed the following calculation:

- 1 instance × $0.0416/hr (on-demand) × 730 hours = **$30.37/month**
- EBS gp3 30 GB × $0.08/GB = **$2.40/month**
- 3,000 IOPS included free with gp3 — no additional IOPS charge
- 125 MBps throughput included free with gp3 — no additional throughput charge

---

## Discounted Pricing (Commitment-Based)

Applying AWS Savings Plans reduces the EC2 compute cost significantly.
Storage and data transfer remain at on-demand rates.

| Commitment Type | EC2 Monthly | Total Monthly | Annual Saving vs On-Demand |
|---|---|---|---|
| On-demand (no commitment) | $30.37 | $43.01 | — |
| 1-year Compute Savings Plan (~36% off) | ~$19.44 | ~$31.08 | ~$143/year |
| 3-year Compute Savings Plan (~60% off) | ~$12.15 | ~$23.79 | ~$231/year |

**Savings Plan flexibility:** AWS Compute Savings Plans apply automatically across
instance families and regions. This means the discount follows the workload even if
the instance type changes in future — making it suitable for growing startups.

---

## Key Observations

1. **Free IOPS and throughput:** EBS gp3 includes 3,000 IOPS and 125 MBps at no
   extra charge — no need to pay for additional performance at this scale.

2. **S3 request costs are negligible:** At $0.09/month, request costs are
   insignificant compared to compute ($30.37) for this workload size.

3. **Commitment discount impact:** A 1-year Savings Plan reduces the monthly bill
   from $43.01 to ~$31.08 — a saving of ~$143/year with no architectural changes.

4. **Nigeria deployment note:** For a West African user base, eu-west-1 (Ireland)
   is recommended over us-east-1 for production deployment. Latency from Lagos to
   Ireland (~150ms) is lower than to N. Virginia (~200ms), at similar or lower cost.
