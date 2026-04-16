---
name: bisection
version: "0.1.0"
description: "Bisection — git bisect, binary search, delta debugging. 'N개 후보 중 원인 하나'를 log N 단계로 격리. regression·장애 구간·최소 재현 입력 탐색."
---

# Bisection

## 한 줄 요약

**원인 공간이 크고 결정론적일 때 이분 탐색**. N개 커밋·N개 입력·N개 구성 중 원인을 `log₂ N` 단계로 찾는다.

## 이론 기원

- **Linus Torvalds** — git bisect (2005). 커널 regression 추적용.
- **Andreas Zeller** — Delta Debugging (1999). 실패 유발 입력 최소화.
- 이분 탐색 알고리즘의 디버깅 응용.

## 세 가지 변형

### 1. Commit Bisection (git bisect)
- "언제부터 깨졌나"
- 명령: `git bisect start / bad / good <ref>`
- 각 단계에서 `good` 또는 `bad` 판정 → 좁혀감
- 자동화: `git bisect run ./test.sh` (exit 0 = good, non-zero = bad)

### 2. Binary Search Input
- "어떤 입력이 트리거하는가"
- 대량 요청 로그·테스트 스위트에서 문제 하나 격리
- 원본 입력의 절반씩 제거하며 실패 여부 확인

### 3. Delta Debugging
- "실패 재현에 *꼭* 필요한 최소 입력"
- 자동화된 최소화 알고리즘 (ddmin)
- HTML 파서 크래시 → 3000줄 페이지를 12글자로 축소한 사례(Zeller)

## 언제 쓰나

- Regression이 있고 good·bad 두 지점 명확할 때
- 테스트는 결정적(deterministic)이어야 함
- 로그·입력·플래그·라이브러리 버전 등 이산 공간 탐색

## 전제 조건

- **결정론** — 같은 조건에서 같은 결과. flaky면 bisect 오작동.
- **단조성(monotonicity)** — 어느 지점 이후 항상 bad (일시적 회복 없음)
- **판정 자동화 가능** — 수동이면 단계당 비용 큼

비결정적이면: 여러 번 실행 + 다수결, 또는 `scientific-debugging`으로 전환.

## 실전 적용

### git bisect 표준 루틴
```
git bisect start
git bisect bad HEAD
git bisect good v1.4.2        # 마지막 정상 릴리즈
git bisect run npm test       # 자동 실행
# 1000 커밋 중 10 단계로 원흉 커밋 찾음
git bisect reset
```

### 판정 스크립트 팁
- exit 0 = good, 1~124/126~127 = bad, 125 = skip(빌드 실패 등)
- 의도치 않은 커밋(빌드 불가)은 `git bisect skip`

### 로그 bisection 예시
```
실패 요청 3000건 중 어느 것이 원인?
→ 1~1500 batch 실행 → 실패 재현? Y → 1~750 → ...
→ 10 라운드에 1건 격리
```

### Delta Debugging 도구
- `creduce` (C/C++)
- `shrinker` (property-based test 내장)
- 파이썬: `pysearch`, 커스텀 스크립트

## 언제 안 쓰나

- **비결정적 실패** — race condition (→ `scientific-debugging`)
- **단조성 깨짐** — 버그가 커밋 A에서 발생, B에서 우연히 숨고, C에서 재발
- **상태 의존** — DB 마이그레이션 포함 커밋 구간 (bisect 시 각 단계 DB 리셋 필요)
- **후보가 작을 때** — 5개 미만이면 순차 확인이 빠름

## 안티패턴

- **테스트 없이 bisect** — 판정이 수동이면 틀림 잦음
- **flaky 테스트로 bisect** — `good`을 `bad`로 잘못 판정하면 전체 오염
- **통합 테스트로 bisect** — 느리고 환경 의존. 가능하면 단위 레벨 재현
- **bisect 완료 후 안주** — 원인 커밋 찾은 게 근본원인 분석 아님

## 한계

1. **규모 큰 병합 커밋** — merge commit이 원흉이면 내부가 다시 bisect 필요
2. **인프라 변화** — 커밋 외 환경 변경(라이브러리, OS) 반영 안 됨
3. **의존 재설치 비용** — node_modules·lockfile 변경 구간에선 단계당 수분 소요

## 이 프레임워크가 *틀렸을 때*

- 비결정·간헐 실패 → `scientific-debugging`
- 운영 중 분산 시스템 이상 → `observability`
- 장애 차단 설계 → `resilience-patterns`

## 추가 학습

- Zeller, A. *Why Programs Fail*, Ch. 13 (Delta Debugging).
- git-scm.com: `git bisect` 문서.
- Regehr, J. *C-Reduce* (블로그 포스트).
