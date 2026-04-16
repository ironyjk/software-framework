---
name: scientific-debugging
version: "0.1.0"
description: "Scientific Debugging — 가설 기반 디버깅. 버그를 미스터리가 아닌 반증 가능한 가설의 연쇄로 취급. Observation→Hypothesis→Experiment→Verification 루프. Zeller *Why Programs Fail* 기반."
---

# Scientific Debugging

## 한 줄 요약

버그는 **반증 가능한 가설의 연쇄**. 관찰 → 가설 → 예측 → 실험 → 검증 → 수정·반복. "느낌"으로 파고들지 말고 *다음 실험으로 무엇이 배제되는지* 항상 의식하라.

## 이론 기원

- **Andreas Zeller** — *Why Programs Fail: A Guide to Systematic Debugging* (2005)
- **David Agans** — *Debugging: 9 Indispensable Rules* (2002)
- 과학적 방법(Popper)의 SW 적용. "확증보다 반증"

## 핵심 루프

```
1. 관찰 (Observation)      — 실패 증상 정확히 기술
2. 가설 (Hypothesis)       — 원인 후보 (검증 가능해야)
3. 예측 (Prediction)       — 가설이 참이면 X가 관찰될 것
4. 실험 (Experiment)       — 최소 비용으로 예측 검증
5. 결과 (Observation)      — 예측 vs 실제
6. 유지/폐기 후 반복        — 가설 트리 좁혀나감
```

## 12 핵심 습관

1. **재현 먼저** — 재현 안 되면 다른 모든 디버깅은 추측
2. **최소 재현** — 제거하면 증상 사라지는 입력까지 축소 (→ delta debugging)
3. **한 번에 한 변수** — 실험은 1변수만 변경
4. **기대치 문서화** — 실행 전 "이러면 X 나올 것" 적고 실제 비교
5. **의심을 코드에** — 프레임워크·컴파일러 버그 의심은 마지막
6. **최근 변경 우선** — regression은 최근 diff에서
7. **로그 더 달기 주저 말기** — 가시성이 가설 비용 낮춘다
8. **스택·상태 덤프 활용** — 메모리 스냅샷, heap dump
9. **Off-by-one 체크리스트** — 경계값(0, 1, max, null, empty, negative)
10. **Rubber duck** — 누군가/무엇에게 설명 = 가설 명료화
11. **디버거 ≠ 무조건 필수** — 가끔 print + logs가 빠르다
12. **해결 후 "왜 발생했나"** — 수정 후 근본원인 문서화 (5 Whys)

## 언제 쓰나

- 재현 가능한 버그의 원인 추적
- "왜 이 시스템이 이렇게 동작하는가" 이해
- 의문 동작(unexpected behavior) 조사
- 성능 저하 원인 탐색 (`observability`와 함께)
- 간헐 실패의 조건 범위 좁히기

## 실전 적용

### 가설 트리 예시
```
증상: 결제가 간헐적 실패
├─ H1: 외부 은행 API 타임아웃
│   실험: 재시도 로그·응답시간 측정
│   → 반증 (API 응답 정상)
├─ H2: DB 커넥션 풀 고갈
│   실험: 풀 metrics 확인
│   → 유지 (풀 사용률 95%+)
│   ├─ H2.1: 쿼리가 느려짐
│   │   실험: slow query log
│   │   → 유지
│   └─ H2.2: 커넥션 누수
│       실험: 명시적 close 누락 탐색
│       → 반증
```

### 핵심: *반증의 비용이 낮은 실험부터*
예: 로그 grep < metric 대시보드 < 재현 환경 재빌드 < 프로덕션 A/B

## 안티패턴

- **Shotgun debugging** — "아무거나 바꿔보자"
- **Heisenberg** — print·logging 추가가 버그 자체 변화시킴 (race condition에서 흔함)
- **Fix-forward panic** — 근본원인 찾기 전 hot fix 연발
- **Confirmation bias** — 자기 가설을 *확증*하는 증거만 수집
- **Premature abstraction** — 원인 모른 채 구조 변경

## 한계

1. **재현 안 되는 버그** — flaky test, race condition에선 가설 검증 비용 급등
2. **분산 시스템** — 원인 분해가 단일 머신처럼 깔끔하지 않음 (→ `observability`)
3. **시간 압박** — 서비스 장애 중엔 "임시방편 먼저, 분석 나중" 택하기도
4. **인지 편향** — 익숙한 코드에선 가설 다양성 떨어짐

## 이 프레임워크가 *틀렸을 때*

- 실패 커밋 추적 필요 → `bisection`
- 분산 시스템 관측 → `observability`
- 장애 연쇄 방지 → `resilience-patterns`

## 추가 학습

- Zeller, A. *Why Programs Fail.*
- Agans, D. *Debugging.* (읽기 빠른 교과서)
- Allspaw, J. *Blameless PostMortems and a Just Culture.* (웹)
