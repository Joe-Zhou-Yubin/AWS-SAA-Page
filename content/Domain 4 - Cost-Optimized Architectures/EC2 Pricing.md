# EC2 Pricing Models

## Overview
Understanding EC2 pricing is critical for Domain 4 (20% of exam). Choose the right model based on workload characteristics.

## Pricing Options

### On-Demand
- Pay by the second (Linux/Windows) or hour (other OS)
- No upfront commitment
- Highest cost per unit
- Use for: short-term, unpredictable workloads, testing

### Reserved Instances (RI)
- 1 or 3-year commitment
- Up to 72% discount vs On-Demand
- Payment options: All Upfront > Partial Upfront > No Upfront
- Types:
  - **Standard RI** — fixed instance type, can sell on marketplace
  - **Convertible RI** — can change instance type/family (lower discount ~66%)
- Scope: Regional or Zonal (capacity reservation)

### Savings Plans
- Commit to $/hour of usage for 1 or 3 years
- **Compute Savings Plan** — any instance family, region, OS, tenancy (up to 66%)
- **EC2 Instance Savings Plan** — specific instance family in a region (up to 72%)
- More flexible than RIs, similar discounts

### Spot Instances
- Up to 90% discount vs On-Demand
- Can be interrupted with 2-minute notice
- Use for: batch jobs, data analysis, distributed workloads, CI/CD
- **Spot Fleet** — mix of Spot + On-Demand to meet target capacity
- Strategies: lowestPrice, diversified, capacityOptimized
- NOT for: databases, critical applications, stateful workloads

### Dedicated Hosts
- Physical server dedicated to you
- Most expensive option
- Use for: licensing (per-socket/per-core), compliance
- Can use RIs on Dedicated Hosts

### Dedicated Instances
- Instances on dedicated hardware (not shared with others)
- May share hardware with other instances in your account
- Less control than Dedicated Hosts

### Capacity Reservations
- Reserve capacity in specific AZ
- No billing discount (combine with RIs/Savings Plans)
- Guarantees capacity when needed

## Decision Framework

| Workload | Best Option |
|----------|-------------|
| Steady-state, predictable | Reserved Instances / Savings Plans |
| Flexible, interruptible batch | Spot Instances |
| Short-term, testing | On-Demand |
| Licensing compliance | Dedicated Hosts |
| Must have capacity guarantee | Capacity Reservations + RI |

## Exam Tips
- Savings Plans are generally preferred over RIs (more flexible)
- Spot + On-Demand mix = cost-effective + reliable
- Spot Instances: know they CAN be interrupted
- Convertible RIs have lower discount but more flexibility
- Dedicated Hosts for software licensing requirements (BYOL)
- AWS Compute Optimizer recommends right-sizing

## SAA Decision Addendum

### Choose This When
- **Savings Plans**: steady compute spend exists and flexibility is important.
- **Standard Reserved Instance**: stable instance family/size/Region requirements and maximum discount matter.
- **Convertible Reserved Instance**: commitment is stable but instance family may change.
- **Spot**: workloads are fault-tolerant, stateless, interruptible, or checkpointed.
- **Capacity Reservation**: capacity in a specific AZ must be guaranteed, with or without a discount instrument.

### Exam Traps
- A discount product is not the same thing as a capacity guarantee.
- Spot is wrong for stateful databases or hard real-time critical paths.
- Dedicated Hosts are for compliance/licensing control; they are usually not cost-optimized unless BYOL drives the requirement.
- Rightsizing often beats buying commitments for overprovisioned resources.

---

Related: [[Domain 3 - High-Performing Architectures/EC2]] | [[Cost Management Tools]]
#aws #ec2 #pricing #cost-optimization #domain4
