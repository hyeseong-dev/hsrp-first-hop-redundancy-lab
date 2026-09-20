# Design Decisions and Implementation Record

## Physical wiring

| Connection | Purpose |
| --- | --- |
| PC1 <-> S3-L2 | client access link |
| S3-L2 <-> S1-L3 | first HSRP member path |
| S3-L2 <-> S2-L3 | second HSRP member path |
| S1-L3 Fa1/11 <-> R1 Fa0/0 | primary upstream path, `192.168.11.0/24` |
| S2-L3 Fa1/12 <-> R1 Fa1/0 | alternate upstream path, `192.168.12.0/24` |

## Decisions made

### One user VLAN first

VLAN 1 deliberately keeps the first question narrow: can the client maintain one gateway address while the active gateway changes? This is a learning simplification, not a production VLAN design.

### HSRP preempt and uplink tracking

S1 priority 110 makes it the normal Active node. Tracking Fa1/11 subtracts 20 on upstream failure, reducing S1 to priority 90. S2 (priority 100) then becomes Active even though S1's VLAN SVI remains alive. S2 also tracks its own uplink with a decrement of 30.

### RIP metric alignment

R1 normally prefers the S1 route to `172.16.1.0/24` (metric 1). S2 advertises that same user subnet with an outbound offset metric of +5, so R1 uses S2 only when S1's route becomes invalid. This prevents normal-path ECMP from obscuring which gateway path is preferred.

### RIP lab timers

All three routing nodes use `timers basic 5 15 15 20`. Default RIP timing makes an interface-side failure take too long to observe in this lab. The shorter values produce a measurable failover window but require a separate production suitability assessment.

## Completed implementation milestones

- HSRP baseline election and virtual gateway reachability
- R1 routed uplinks, RIPv2, and Loopback0 reachability target
- HSRP uplink tracking with priority decrement
- R1 return-path preference and automatic alternate-path selection
- Active-node fault, recovery, and preempt validation

## Deferred scope: MHSRP

The single-VLAN baseline keeps this work intentionally separate. Its implementation is now available in the `feat/mhsrp-vlan-expansion` branch and consists of VLAN 10/VLAN 20 SVIs, two virtual gateways, dot1q uplinks, split Active roles, and aligned RIPv2 return paths.

## MHSRP implementation note

The expansion uses S1 as the normal Active gateway for VLAN 10 and S2 for VLAN 20. This distributes normal first-hop forwarding without adding GLBP. On an S1 uplink fault, S2 assumes VLAN 10 as well; after recovery, S1 preempts only VLAN 10 and the intended split returns.

The HSRP group number is not required to equal the VLAN ID. VLAN 10 uses group 1 because group 10's virtual MAC was not reachable in this specific GNS3/IOS combination. VLAN 20 retains group 20. The saved runtime configuration under `configs/mhsrp/` is authoritative for this emulator-specific decision.
