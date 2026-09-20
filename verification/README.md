# Verification Record

## Commands used

On S1-L3 and S2-L3:

```text
show standby brief
show standby vlan 1
show track
show ip route
```

On R1:

```text
show ip route 172.16.1.0
show ip protocols
```

On PC1:

```text
ping 172.16.1.254 -c 5
ping 192.0.2.1 -c 5
```

## Observed results

| Test | Actual result | Status |
| --- | --- | --- |
| Initial election | S1-L3 Active, priority 110; S2-L3 Standby, priority 100 | pass |
| Default gateway | PC1 ping to `172.16.1.254` returned 5/5 | pass |
| Upstream target | PC1 ping to R1 Loopback0 `192.0.2.1` returned 5/5 after convergence | pass |
| Normal return path | R1 used S1 `192.168.11.1`, RIP metric 1 | pass |
| Fault injection | `shutdown` on S1-L3 Fa1/11 | executed |
| HSRP failover | S1-L3 became Standby at effective priority 90; S2-L3 became Active | pass |
| Return-path failover | R1 used S2 `192.168.12.2`, RIP metric 6 | pass |
| Recovery | `no shutdown` on S1 Fa1/11 returned S1 to Active through preempt; S2 returned Standby | pass |

## Interpretation

The test proves the intended relationship between first-hop selection and upstream return routing. It does **not** prove a lossless network: one remote probe was lost during recovery, which is expected while HSRP and RIP reconverge. Record packet loss and control-plane timing in each future topology change.
