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

## 단일 HSRP 실제 결과

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

## MHSRP 확장 검증

기능 브랜치는 VLAN 10/20과 대표 단말 2대를 추가했습니다. 정상 상태에서 S1은 VLAN 10 Active, S2는 VLAN 20 Active입니다. 두 단말은 자신의 가상 IP와 R1 Loopback0에 도달했고, R1도 각 VLAN의 Active 장비를 반환 경로로 선택했습니다.

| 검증 항목 | 실제 결과 | 상태 |
| --- | --- | --- |
| 정상 Active 분산 | VLAN 10: S1 Active / VLAN 20: S2 Active | 성공 |
| VLAN 10 단말 | PC10 → `172.16.10.254`, R1 Loopback0 통신 성공 | 성공 |
| VLAN 20 단말 | PC20 → `172.16.20.254`, R1 Loopback0 통신 성공 | 성공 |
| S1 uplink 장애 | S1 유효 우선순위: VLAN 10은 90, VLAN 20은 70 | 성공 |
| 장애 상태 Active | S2가 두 VLAN의 Active 역할 수행 | 성공 |
| VLAN 10 반환 경로 | R1이 S2로 전환, RIP metric 6 | 성공 |
| 복구 후 분산 | S1은 VLAN 10 Active 복귀, S2는 VLAN 20 Active 유지 | 성공 |
