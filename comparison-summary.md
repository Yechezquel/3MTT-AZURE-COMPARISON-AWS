# Cost Comparison Summary — AWS vs Azure

## Overview

This report compares the monthly cost of hosting a small Linux web application
on Amazon Web Services (AWS) and Microsoft Azure. The same application
specification was configured on both platforms using their official pricing
calculators, with identical compute, storage, and network requirements.

---

## Master Comparison Table

| Scenario | Monthly Cost | Annual Cost | vs AWS Baseline |
|---|---|---|---|
| **AWS** Linux t3.medium (on-demand) | $43.01 | $516.12 | Baseline |
| **Azure** Linux B2s (pay-as-you-go) | $32.77 | $393.22 | -$10.24/mo cheaper |
| **Azure** Windows B2s (no AHB) | ~$65.86 | ~$790.32 | +$22.85/mo more expensive |
| **Azure** Windows B2s + AHB | $32.77 | $393.22 | -$10.24/mo cheaper |

> **Finding:** Azure Linux is $10.24/month cheaper than AWS on-demand.
> However, AWS includes 100 GB free egress while Azure charges ~$8.27 for
> the same volume. Adjusting for egress, the true gap is ~$1.97/month.

---

## Service-by-Service Breakdown

| Component | AWS Service | AWS Cost | Azure Service | Azure Cost |
|---|---|---|---|---|
| Compute (2 vCPU, 4 GB, 730 hrs) | EC2 t3.medium | $30.37 | VM B2s | $30.37 |
| OS disk (30–32 GB SSD) | EBS gp3 | $2.40 | Managed Disk E4 | $2.40 |
| Object storage (50 GB) | S3 Standard | $1.15 | Blob Storage LRS | $1.04 |
| Storage requests | S3 API calls | $0.09 | Blob operations | $0.00 |
| Egress (100 GB) | Data transfer out | $9.00 | Bandwidth | ~$8.27* |
| **Total** | | **$43.01** | | **$32.77–$41.04** |

> *Azure egress was configured within the VM block and showed $0.00 in the
> estimate. The true egress cost for 95 GB (after 5 GB free tier) is ~$8.27.

---

## Networking Cost Analysis

### Cross-Zone Data Transfer

Both AWS and Azure charge **$0.01/GB per direction** for data transferred
between Availability Zones within the same region.

For our single-AZ architecture (Pattern B), cross-zone transfer cost = **$0.00**
on both platforms. No cross-AZ traffic is generated.

**If the architecture used multiple AZs** (Pattern C — 500 GB/month internal traffic):

| Platform | Cross-AZ rate | 500 GB round-trip cost |
|---|---|---|
| AWS | $0.01/GB per direction | $10.00/month |
| Azure | $0.01/GB per direction | $10.00/month |

Both platforms charge identically for cross-AZ transfer.

### Internet Egress Comparison

The significant networking difference lies in the free tier:

| Platform | Free egress/month | Cost for 100 GB | Cost per GB above free tier |
|---|---|---|---|
| AWS | **100 GB** | $0.00–$9.00 | $0.09/GB |
| Azure | **5 GB** | ~$8.27 | $0.087/GB |

AWS's generous 100 GB free tier is a clear advantage for small web apps
with moderate traffic. For workloads exceeding 1 TB/month, Azure's lower
per-GB rate ($0.087 vs $0.09) becomes more favourable.

---

## Discount Mechanisms

### AWS Savings Plans

AWS Compute Savings Plans apply a discount in exchange for a committed
hourly spend over 1 or 3 years. The discount applies automatically across
instance families and regions — no lock-in to a specific VM type.

For the t3.medium workload ($30.37/month compute):

| Commitment | Monthly compute | Saving vs on-demand | Annual saving |
|---|---|---|---|
| On-demand | $30.37 | — | — |
| 1-year Savings Plan (~36% off) | ~$19.44 | ~$10.93/mo | ~$131/yr |
| 3-year Savings Plan (~60% off) | ~$12.15 | ~$18.22/mo | ~$219/yr |

### Azure Reserved Instances

Azure Reservations commit to a specific VM size for 1 or 3 years in exchange
for a discount on the compute rate. Can be exchanged for equal or higher value
reservations. Early cancellation carries a 12% termination fee.

For the B2s workload ($30.37/month compute):

| Commitment | Monthly compute | Saving vs pay-as-you-go | Annual saving |
|---|---|---|---|
| Pay-as-you-go | $30.37 | — | — |
| 1-year Reserved (~42% off) | ~$17.62 | ~$12.75/mo | ~$153/yr |
| 3-year Reserved (~55% off) | ~$13.67 | ~$16.70/mo | ~$200/yr |

### AHB + Reservation Stacking (Azure advantage)

Azure allows Azure Hybrid Benefit and Reservations to apply simultaneously.
For a Windows workload:

- AHB removes the Windows licence cost (~$32/month saving)
- 1-year Reservation then discounts the compute by a further 42%
- Combined saving vs pay-as-you-go Windows: **over 70%**

AWS has no equivalent to AHB — there is no platform-level mechanism to
bring existing Windows licences and receive a discount.

---

## Cost-Optimisation Strategies

### Strategy 1 — Commitment-Based Discounts

**Action:** Move from on-demand/pay-as-you-go pricing to a 1-year commitment.

- **AWS:** Apply a 1-year Compute Savings Plan → saves ~$131/year on compute alone
- **Azure:** Apply a 1-year Reservation → saves ~$153/year on compute alone
- **Rule:** Only commit to baseline usage. Keep burst or experimental capacity on-demand.
- **When to do it:** Once the application has been running stably for 2–3 months
  and usage patterns are predictable.

### Strategy 2 — Right-Sizing and Single-AZ Architecture

**Action:** Choose the smallest instance that meets performance requirements and
keep all resources in a single Availability Zone.

- **Right-sizing example:** A t3.small (2 vCPU, 2 GB RAM) costs ~$15/month vs
  t3.medium at $30.37/month. For a low-traffic app, this halves the compute bill.
- **Single-AZ benefit:** Eliminates cross-AZ transfer charges ($0.01/GB/direction).
  At 500 GB/month internal traffic, this saves $10/month on both platforms.
- **Static asset offloading:** Serve images and CSS/JS from S3/Blob Storage directly
  rather than through the VM. This reduces VM CPU load and egress billed at VM rates.

---

## Provider Recommendation by Scenario

### Scenario A — Linux-based Nigerian startup

**Recommended provider: AWS**

On-demand, AWS costs $43.01/month vs Azure's $32.77/month — but Azure's lower
figure does not account for the $8.27 egress gap. Adjusting for egress, Azure
is ~$1.97/month cheaper. However, AWS's Compute Savings Plans offer greater
flexibility for a startup that may change instance types as it scales.

For a Nigeria-based team, **eu-west-1 (Ireland)** is the recommended AWS region
for production — offering ~150ms latency from Lagos at approximately 30% lower
cost than af-south-1 (Cape Town).

### Scenario B — Microsoft-centric enterprise

**Recommended provider: Azure**

An organisation with existing Windows Server and SQL Server licences covered
by Software Assurance should use Azure. Azure Hybrid Benefit eliminates the
Windows licence cost, and combined with a 1-year Reservation, saves over 70%
versus pay-as-you-go Windows pricing. AWS has no equivalent mechanism.

The ecosystem advantage reinforces this: Azure Active Directory, Microsoft 365
integration, and unified compliance tooling reduce operational overhead for
Microsoft-centric organisations.

---

## Calculator Evidence

| Platform | Calculator URL |
|---|---|
| AWS | https://calculator.aws/#/estimate?id=ecb1d72711d2b3eae96c16334a9262098f606673 |
| Azure | https://azure.com/e/8cb758dfe9be487db3bc38cf914d1f3e |

Screenshots of configured calculators are available in the `/screenshots/` directory.
