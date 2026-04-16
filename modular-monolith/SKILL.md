---
name: modular-monolith
version: "0.1.0"
description: "Modular Monolith — 단일 배포 단위 안에 모듈 경계를 엄격하게 두는 아키텍처. Shopify·Basecamp 사례. 마이크로서비스의 복잡도 없이 경계의 이익. 미래 분리의 안전한 출발점."
---

# Modular Monolith

## 한 줄 요약

**하나로 배포, 안은 마이크로처럼 쪼갠 모노**. 배포·트랜잭션·디버깅은 단순(모노의 장점), 코드·팀 경계는 명확(마이크로의 장점). 대부분의 스타트업·제품의 *기본값이어야 함*.

## 이론 기원

- **DHH / Basecamp** — "The Majestic Monolith" (Signal v. Noise 블로그, 2016). "Majestic Monolith" 용어의 출처.
- **Shopify (Kirsten Westeinde)** — "Deconstructing the Monolith" (shopify.engineering, 2019). Rails 모놀리스 → 모듈러 모노 전환 사례.
- **Simon Brown** — "Modular Monolith" 용어 대중화.
- **Sam Newman** — *Monolith to Microservices* (O'Reilly, 2019). 점진 분리 전략.

## 핵심 원칙

1. **한 프로세스·한 배포** — 모놀리스의 단순성 유지
2. **모듈 경계 강제** — 외부로 노출되는 API만 노출, 내부 구현 숨김
3. **공유 DB 가능, 공유 테이블 금지** — 모듈별 스키마/소유권 명확
4. **모듈 간 통신 = 명시적 인터페이스** — 함수 호출이지만 계약은 서비스 수준
5. **나중에 쪼갤 수 있게** — 각 모듈이 추후 별도 서비스로 뽑힐 수 있는 경계

## 왜 마이크로서비스보다 먼저

| 항목 | 모노 | 모듈러 모노 | 마이크로서비스 |
|---|---|---|---|
| 배포 복잡도 | 낮음 | 낮음 | 높음 |
| 트랜잭션 | 간단 | 간단 | 분산 (어려움) |
| 디버깅 | 로컬 스택 | 로컬 스택 | 분산 추적 필수 |
| 경계 규율 | 느슨 | 강제 가능 | 강제됨 |
| 팀 독립성 | 낮음 | 중간 | 높음 |
| 운영 비용 | 낮음 | 낮음 | 높음 |
| 독립 스케일링 | 불가 | 불가 | 가능 |

*초기·중기 스타트업의 비용 대비 이익*: Modular Monolith가 대부분 승.

## 모듈 경계 강제 방법

### 언어 레벨
- Java: module-info.java (Java 9+)
- Rust: crate 경계
- Python: 패키지 구조 + 린터 (import-linter)
- TypeScript: workspace 패키지 + lint 규칙
- Ruby: packwerk (Shopify 오픈소스)

### DB 레벨
- 모듈별 스키마 분리
- 외부 모듈은 해당 스키마 읽기·쓰기 금지
- 조인 필요 → 내부 API 호출

### 코드 리뷰 레벨
- "이 모듈은 저 모듈을 모르도록" 리뷰어 원칙
- 순환 의존 탐지 도구

## 언제 쓰나

- 스타트업 MVP ~ 중기 제품
- 팀 1~30명 규모
- 도메인이 한두 개 제품 중심
- 아직 독립 스케일링 요구 없음
- **기본값으로 선택**

## 마이크로서비스로 언제 쪼개나 (쪼개지 않을 이유부터)

쪼갤 **정당한 이유**:
1. 팀 독립 배포 강하게 필요 (팀 50명+ 등)
2. 특정 모듈이 독립 스케일링 필요 (다른 모듈의 10배 부하)
3. 서로 다른 기술 스택·런타임이 정말 필요
4. 조직 분리 (M&A, 스핀오프)

쪼개지 말아야 할 이유:
1. "마이크로서비스가 멋있으니까" — 1순위 실패 사유
2. "코드 베이스가 커서" — 모듈화로 해결
3. "성능 문제" — 일단 프로파일링부터
4. "이 팀이 독립하고 싶어서" — 경계부터 정리

## 실전 적용

### Shopify 사례
- Rails 모놀리스 유지, ~수백만 줄
- `packwerk`로 모듈 경계 검사
- 공개 API 존재: Merchants, Products, Orders 등
- *이 규모에서도* majestic monolith

### 점진 도입
1. 현 모노의 암묵적 경계 식별
2. 모듈 경계 후보 선정 (bounded context ≈ 모듈)
3. 의존 그래프 그리고 순환 제거
4. 모듈별 코드 이동 + 명시 API
5. 린터·CI로 경계 위반 차단

## 안티패턴

- **이름만 모듈러** — 경계 없음, 여전히 얽힘
- **DB는 공유, 조인은 마구** — 진짜 경계 아님
- **모듈 너무 많음** — 20개 이상 되면 관리 불가. 5~10이 보통
- **"미래에 뽑을 거니까" 과설계** — YAGNI
- **마이크로로 성급 분리** — 모노에서도 분리 못 하던 팀은 마이크로에서도 못 함

## 한계

1. **단일 배포 장애 반경** — 한 모듈 배포 실수가 전체 영향
2. **단일 런타임 제약** — 같은 언어·프레임워크 강제
3. **독립 스케일링 불가** — CPU·메모리 모듈별 할당 못함
4. **팀 수 늘면 배포 조율 비용** — 20명+ 이상에서 병목

## 이 프레임워크와 함께 쓰는 것들

- `ddd` — bounded context = 모듈
- `hexagonal` — 모듈 내부 구조
- `team-topologies` — 팀과 모듈 정렬
- `strangler-fig` — 레거시 모노 → 모듈러 모노 전환

## 이 프레임워크가 *틀렸을 때*

- 팀 100명+ 독립 배포 필수 → 마이크로 분리
- 이미 모듈 경계 명확·운영 성숙 → 마이크로 후보
- 극단적 다른 기술 스택 필요 → 분리

## 추가 학습

- Shopify Engineering. "Deconstructing the Monolith" (블로그).
- Newman, S. *Monolith to Microservices.* (2019)
- Brown, S. *Software Architecture for Developers.* — modular monolith 섹션.
- Packwerk (github.com/Shopify/packwerk) — Ruby 모듈 경계 검사 도구.
