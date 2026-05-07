# Cloud Cost Comparison — AWS vs Azure

A comprehensive cost comparison between Amazon Web Services (AWS) and Microsoft
Azure for hosting a small Linux web application.

## Project Overview

This project analyzes the monthly cost of running a small web application on
both AWS and Azure, covering compute, storage, networking, licensing impact
(Azure Hybrid Benefit), and commitment-based discount mechanisms.

**Application spec:** 2 vCPU, 4 GB RAM, 30 GB SSD, 50 GB object storage, 100 GB egress/month
**Region:** AWS us-east-1 (N. Virginia) / Azure East US

---

## Repository Structure

```
cloud-cost-comparison/
├── README.md
├── summary.txt                        ← 300-word business justification
├── report/
│   ├── aws-cost-breakdown.md          ← AWS line-item cost breakdown
│   ├── azure-cost-breakdown.md        ← Azure breakdown (Linux + Windows + AHB)
│   └── comparison-summary.md         ← Master comparison table + analysis
├── estimates/
│   ├── aws-estimate-link.txt          ← AWS Pricing Calculator URL
│   └── azure-estimate.xlsx            ← Exported Azure estimate
└── screenshots/
    ├── aws-ec2-config.png
    ├── aws-s3-config.png
    ├── aws-estimate-summary.png
    ├── azure-vm-linux-config.png
    ├── azure-vm-windows-ahb.png
    ├── azure-blob-config.png
    └── azure-estimate-summary.png
```

---

## Key Findings

| Scenario | Monthly Cost |
|---|---|
| AWS Linux t3.medium (on-demand) | $43.01 |
| Azure Linux B2s (pay-as-you-go) | $32.77 |
| Azure Windows B2s (no AHB) | ~$65.86 |
| Azure Windows B2s + Hybrid Benefit | $32.77 |

- **AWS** provides a more generous egress free tier (100 GB vs Azure's 5 GB)
- **Azure Hybrid Benefit** eliminates the Windows licence cost — saving ~$397/year
- Both platforms reach comparable costs when 3-year commitments are applied

---

## Calculator Links

- **AWS Estimate:** https://calculator.aws/#/estimate?id=ecb1d72711d2b3eae96c16334a9262098f606673
- **Azure Estimate:** https://azure.com/e/8cb758dfe9be487db3bc38cf914d1f3e

---

## Recommendation

- **Linux startup** → AWS (Compute Savings Plans, flexible discounts)
- **Microsoft enterprise** → Azure (Hybrid Benefit + Reservations = 70%+ saving on Windows)
