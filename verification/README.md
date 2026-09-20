# 검증 기록

## 사용한 검증 명령

S1-L3 및 S2-L3:

```text
show standby brief
show standby vlan 1
show track
show ip route
```

R1:

```text
show ip route 172.16.1.0
show ip protocols
```

PC1:

```text
ping 172.16.1.254 -c 5
ping 192.0.2.1 -c 5
```

## 실제 결과

| 검증 항목 | 실제 결과 | 상태 |
| --- | --- | --- |
| 초기 선출 | S1-L3 Active, 우선순위 110 / S2-L3 Standby, 우선순위 100 | 성공 |
| 기본 게이트웨이 | PC1 → `172.16.1.254`: 5/5 응답 | 성공 |
| 상단 검증 대상 | PC1 → R1 Loopback0 `192.0.2.1`: 수렴 후 5/5 응답 | 성공 |
| 정상 반환 경로 | R1이 S1 `192.168.11.1`, RIP metric 1 선택 | 성공 |
| 장애 주입 | S1-L3 Fa1/11에서 `shutdown` | 실행 |
| HSRP 장애 전환 | S1-L3 Standby(유효 우선순위 90), S2-L3 Active | 성공 |
| 반환 경로 전환 | R1이 S2 `192.168.12.2`, RIP metric 6 선택 | 성공 |
| 복구 | S1 Fa1/11 `no shutdown` 후 S1이 preempt Active 복귀, S2 Standby 복귀 | 성공 |

이 결과는 첫 번째 홉 선택과 상단 반환 경로가 함께 전환되는 것을 증명합니다. 다만 무손실 네트워크를 의미하지는 않습니다. 복구 중 원격 probe 1건이 손실됐으며, 이후 변경에서도 패킷 손실과 제어 영역 수렴 시간을 기록해야 합니다.
