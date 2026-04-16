---
name: strangler-fig
version: "0.1.0"
description: "Strangler Fig Pattern (Martin Fowler) — 레거시를 한 번에 갈아엎지 말고 주변에 새 시스템을 키워 점진 교체. Branch by Abstraction·Parallel Change 포함. 리라이트 실패의 대안."
---

# Strangler Fig Pattern

## 한 줄 요약

**옛 나무를 잘라내지 말고, 옆에 무화과나무를 심어 천천히 감싸 죽여라.** 레거시 시스템을 한 번에 리라이트하지 말고, 기능 단위로 새 시스템이 옛것을 대체하게 만든다.

## 이론 기원

- **Martin Fowler** — "StranglerFigApplication" (2004). 호주 여행 중 본 strangler fig(교살무화과) 비유.
- 대안 이름: Strangler Pattern, Gradual Migration.
- 함께 쓰는 기법: **Branch by Abstraction** (Paul Hammant), **Parallel Change / Expand-Contract** (Danilo Sato).
- 뿌리: 1990s "리라이트는 거의 실패한다"는 Joel Spolsky의 주장에 대한 실무적 답.

## 왜 리라이트가 실패하는가

- 비즈니스는 멈추지 않음 (신기능 계속)
- 원 시스템 코드에 *암묵 지식* (엣지케이스·히스토리)
- 완전 교체 = Big Bang = 실패하면 롤백 못함
- 팀 사기 붕괴 (2년째 새로 만드는데 아직 안 나옴)

## Strangler의 전략

### 단계
```
1. 경계 식별: 교체할 기능·모듈 선정 (작고 독립적인 것부터)
2. 래퍼(Facade): 옛 시스템 앞에 라우팅 레이어
3. 신규 구현: 새 서비스·모듈이 해당 기능 제공
4. 트래픽 이관: 일부 → 전부 점진 전환
5. 옛 코드 삭제
6. 반복
```

### 함께 쓰는 보조 기법

**Branch by Abstraction** (Hammant)
- 레거시 위에 *추상 인터페이스* 생성
- 기존 구현을 그 인터페이스 뒤로
- 새 구현을 같은 인터페이스로 추가
- 플래그로 전환
- 옛 구현 제거

**Parallel Change (Expand-Contract)** (Sato)
- Expand: 새 버전 추가, 옛 버전 유지
- Migrate: 호출자·데이터를 점진 이관
- Contract: 옛 버전 제거

**Feature Toggle**
- 신규 경로 vs 옛 경로 런타임 스위치
- 카나리 전환 (1% → 10% → 50% → 100%)
- 문제 시 즉시 롤백

## 언제 쓰나

- 레거시 모놀리스 → 마이크로·모듈러 모노 전환
- 기술 스택 교체 (PHP→Go, JSP→React)
- 데이터 저장소 마이그레이션 (RDB→NewSQL, 단일 DB→샤딩)
- 외부 API 버전 업그레이드
- 코드베이스 분할 (회사 분사·M&A)

## 실전 적용

### 트래픽 라우팅 위치
- **Ingress / API Gateway** — 가장 깔끔. path·header 기반 분기.
- **Load Balancer** — 가중치 라우팅
- **Service Mesh** — Istio VirtualService 등
- **애플리케이션 레이어** — 레거시 내부에 분기 로직 (최후)

### 데이터 이관 패턴
- **Dual Write**: 새·옛 DB 둘 다 쓰기 (일관성 검증)
- **CDC (Change Data Capture)**: 옛 DB 변경을 새 DB로 복제
- **Outbox Pattern**: 이벤트로 이관
- 데이터 소유권 명확히: 한 도메인은 한 곳만 authoritative

### 단계별 체크포인트
- [ ] 경계가 명확한가
- [ ] 카나리 1% 24시간 통과
- [ ] 롤백 절차 자동화
- [ ] 옛·새 결과 비교 로그 (shadow traffic)
- [ ] SLO 유지 확인

## 안티패턴

- **경계 없는 strangler** — 이건 그냥 리라이트의 느린 버전. 경계 없으면 옛 시스템이 안 죽음.
- **Big Bang in Strangler 옷** — 95% 구축 후 한 번 전환 = 여전히 Big Bang
- **옛 코드 삭제 안 함** — 두 시스템 유지비 = 지옥. 삭제가 플랜의 핵심.
- **신규에서 같은 실수** — 아키텍처·테스트 규율 없는 신규는 새 레거시
- **데이터 이관 후회** — 나중에 "어? 옛 DB도 쓰고 있었네" 발견

## 한계

1. **기간 김** — 진짜 strangler는 1~5년. 리더십 인내 필요.
2. **이중 운영 비용** — 이관 기간 동안 두 시스템 다 운영
3. **초기 속도 느림** — 경계·라우팅 세팅이 선행
4. **일부 영역 기술적 불가능** — 데이터 모델 변경이 매우 큰 경우 strangler 어려움
5. **심리적 저항** — "이럴 거면 그냥 리라이트" 주장이 계속 나옴

## 성공 조건

1. **리더십이 장기 플랜을 보장**
2. **"옛 코드 삭제" 마일스톤 명시** — 없으면 영구 동거
3. **팀이 두 시스템 이해** — 신규만 아는 팀은 실패
4. **경계 선정 안목** — 첫 타깃은 *작고 독립·가시적 가치*
5. **카나리·롤백 자동화**

## 한국 레거시 현장 맥락

- **PHP 5.x/7.x 이커머스 모놀리스** — 10~15년 된 대형몰·티몬·11번가 계열 전형. MySQL FK·트리거·저장 프로시저 얽힘이 strangler 최대 적.
- **JSP + Struts + iBATIS 금융·공공** — 2000s 구축, 벤더 락인. 망분리·전자서명·공인인증 의존 때문에 리라이트보다 strangler가 현실적.
- **SI 벤더 인수인계 코드** — 문서 없음·테스트 없음·오너 없음. strangler 시작 전 *코드 고고학*(git blame + 인터뷰) 필요.
- **공공 클라우드 전환 (행안부·KISA 가이드)** — 클라우드 네이티브 전환 의무화 추세. strangler + `twelve-factor`로 단계 접근.
- **금융권 "차세대 프로젝트"** — 2~3년 대규모 재구축 관례 — 여전히 빅뱅 성향. 최근 카카오뱅크·토스뱅크 등 스타트업 모델이 점진 접근 영향.
- **경영진 "마이크로서비스 3년 전환" 공표** — 정치적으로 빅뱅 금지는 받지만 *종착지가 마이크로여야 하는가*는 별도 논의. 모듈러 모노 + 선별 마이크로 하이브리드 제안이 현실적.

## 이 프레임워크와 함께 쓰는 것들

- `hexagonal` — 신규 경계에 Ports & Adapters 적용
- `ddd` — 경계 선정의 이론적 근거 (bounded context)
- `modular-monolith` — 마이크로 가기 전 중간 목적지
- `observability` — 이관 중 신·구 비교·SLO 확인

## 이 프레임워크가 *틀렸을 때*

- 정말 작은 시스템 → 그냥 리라이트가 빠름
- 스키마가 근본 잘못돼 호환 불가능 → 새 DB 병행 운영 필요
- 규제·계약상 "지금 전면 교체" 강제

## 추가 학습

- Fowler, M. "StranglerFigApplication" (martinfowler.com)
- Newman, S. *Monolith to Microservices.* Ch. 4.
- Hammant, P. "Branch by Abstraction" (블로그)
- Sato, D. "ParallelChange" (martinfowler.com)
