# Azure Cost Breakdown — Small Web Application

## Application Specification

The same Linux-based web application as the AWS comparison, using identical
compute, storage, and network requirements for a fair side-by-side analysis.
An additional Windows Server scenario is included to demonstrate the impact
of Azure Hybrid Benefit.

### Compute
- **vCPU:** 2 cores
- **RAM:** 4 GB
- **Operating System:** Ubuntu (Linux) — and Windows Server with Hybrid Benefit
- **Uptime:** 730 hours/month (24/7)

### Storage
- **OS/App disk:** 32 GB Standard SSD (Azure E4 Managed Disk — closest to 30 GB)
- **Object storage:** 50 GB Blob Storage LRS
- **Monthly API requests:** 100,000 read + 10,000 write operations

### Network
- **Monthly egress:** 100 GB
- **Ingress:** Free on Azure
- **Architecture:** Single AZ — Pattern B (single VM + object storage, no load balancer)

### Region
- **Azure Region:** East US
- **Rationale:** Equivalent reference region to AWS us-east-1 for cost comparison.
  For a Nigeria-based production deployment, West Europe (Netherlands) would offer
  better latency from Lagos at lower cost than South Africa North (Johannesburg).

---

## Azure Services Used

| Azure Service | Purpose | Equivalent Need |
|---|---|---|
| Virtual Machine B2s | Compute | 2 vCPU, 4 GB RAM |
| Managed Disk E4 Standard SSD | Block storage | OS disk and application code |
| Azure Blob Storage LRS | Object storage | Static assets and backups |
| Bandwidth (Internet Egress) | Networking | Internet egress to end users |

---

## Scenario A — Linux (Pay-as-you-go)

| Line Item | Configuration | Monthly Cost |
|---|---|---|
| VM B2s | Linux, 730 hrs, pay-as-you-go, East US | $30.37 |
| Managed Disk E4 | 32 GB Standard SSD | $2.40 |
| Blob Storage LRS | 50 GB, Block Blob, Hot tier | $1.04 |
| Storage transactions | Read/write operations | $0.00 |
| Bandwidth | 100 GB internet egress | $0.00* |
| **Total (Linux, Pay-as-you-go)** | | **$32.77/month** |

> *Bandwidth was configured within the VM block. See networking note below.

---

## Scenario B — Windows Server (No Azure Hybrid Benefit)

| Line Item | Configuration | Monthly Cost |
|---|---|---|
| VM B2s | **Windows**, 730 hrs, pay-as-you-go | $62.42 |
| Managed Disk E4 | 32 GB Standard SSD | $2.40 |
| Blob Storage LRS | 50 GB | $1.04 |
| Bandwidth | 100 GB egress | $0.00* |
| **Total (Windows, No AHB)** | | **~$65.86/month** |

> Without AHB, the Windows VM costs approximately **$32.05 more per month**
> than the Linux VM — this is the embedded Windows Server licence cost.

---

## Scenario C — Windows Server + Azure Hybrid Benefit ✅

| Line Item | Configuration | Monthly Cost |
|---|---|---|
| VM B2s | Windows + **AHB applied**, 730 hrs | $30.37 |
| Managed Disk E4 | 32 GB Standard SSD | $2.40 |
| Blob Storage LRS | 50 GB | $1.04 |
| Bandwidth | 100 GB egress | $0.00* |
| **Total (Windows + AHB)** | | **$32.77/month** |

> **Azure Hybrid Benefit confirmed working:** The Windows + AHB VM costs
> exactly the same as the Linux VM ($32.77/month). AHB removes the Windows
> Server licence cost entirely, reducing the VM rate from ~$62/month to $30.37/month.

---

## Impact of Azure Hybrid Benefit

| Scenario | Monthly Cost | vs Linux Baseline |
|---|---|---|
| Linux | $32.77 | Baseline |
| Windows — no AHB | ~$65.86 | +$33.09/month (+101%) |
| Windows + AHB | $32.77 | **Same as Linux** ✅ |

**Annual saving from applying AHB:** ~$397/year
**Requirement:** Existing Windows Server licences with Software Assurance coverage.

---

## Full Estimate Total (Both Scenarios Combined)

The Azure calculator showed a combined total for all configured services:

| | Amount |
|---|---|
| Estimated monthly cost (all services) | **$67.98/month** |
| Estimated annual cost | **$815.76/year** |

> Note: This combined total includes both the Linux VM and Windows+AHB VM
> simultaneously. For a single-scenario deployment, the per-scenario cost
> is **$32.77/month** (Linux or Windows+AHB).

> **Azure Pricing Calculator estimate:**
> https://azure.com/e/8cb758dfe9be487db3bc38cf914d1f3e

---

## Networking Note

Azure's free egress tier covers only the first **5 GB/month**, compared to
AWS's 100 GB/month free tier. For this workload (100 GB egress):

| Platform | Free egress tier | Billed egress | Egress cost |
|---|---|---|---|
| AWS | 100 GB/month | 0 GB | $0.00 |
| Azure | 5 GB/month | 95 GB | ~$8.27 |

This $8.27 difference is the primary driver of the cost gap between the two
platforms at this traffic level. Above 100 GB/month, AWS begins charging
$0.09/GB while Azure charges $0.087/GB — at high volumes the gap narrows.

---

## Discounted Pricing (Commitment-Based)

Azure Reservations reduce the VM compute cost. Storage and bandwidth remain
at pay-as-you-go rates.

| Commitment | VM Monthly | Total Monthly | Annual Saving |
|---|---|---|---|
| Pay-as-you-go | $30.37 | $32.77 | — |
| 1-year Reserved (~42% off) | ~$17.62 | ~$20.02 | ~$153/year |
| 3-year Reserved (est. ~55% off) | ~$13.67 | ~$16.07 | ~$200/year |

**AHB + Reservation stacking:** For Windows workloads, applying both AHB and
a 1-year Reservation simultaneously achieves the maximum possible discount.
The Reservation reduces compute cost by ~42% on top of AHB already removing
the licence cost — resulting in savings exceeding 70% versus pay-as-you-go
Windows pricing.

---

## Key Observations

1. **AHB is transformative for Windows workloads:** It reduces a Windows VM
   from ~$65.86/month to $32.77/month — saving ~$397/year on a single VM.
   Any Microsoft enterprise with Software Assurance should always apply AHB.

2. **Linux costs are competitive with AWS:** At $32.77/month, Azure Linux is
   actually slightly cheaper than AWS ($43.01/month) for compute + storage alone
   when egress is excluded from the comparison.

3. **Egress free tier is Azure's main disadvantage:** The 5 GB free tier vs
   AWS's 100 GB free tier adds ~$8.27/month at this traffic level. This gap
   disappears for high-traffic workloads where both platforms charge per-GB.

4. **Reservations + AHB stack:** Unlike AWS (which has no AHB equivalent),
   Azure allows two independent discounts to apply simultaneously — making
   Azure the clear winner for Microsoft-licensed enterprise workloads.
