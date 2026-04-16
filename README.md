# Software Framework

소프트웨어 엔지니어링 메타 라우터 + 프레임워크 컬렉션. 디버깅·아키텍처·설계·리질리언스·진화·팀 구조 12가지를 하나의 라우팅 레이어에 묶는다.

## 설계 원칙

1. **프랙티스보단 식별 가능한 프레임워크** — "잘 하자"가 아니라 "무엇을 본다"가 명확한 도구만.
2. **수명 긴 원칙 + 최근 실무** — Fowler·Evans·Cockburn 같은 고전 + Team Topologies·Resilience 패턴 등 현장.
3. **과적용 경고 포함** — 각 프레임워크가 틀리거나 해로울 때를 명시.

## 구조

```
software-framework/
├── SKILL.md                # 메타 라우터
├── scientific-debugging/   # 가설 기반 디버깅
├── bisection/              # git bisect · binary search · delta
├── observability/          # USE + RED + 4 Golden Signals
├── hexagonal/              # Ports & Adapters (Cockburn)
├── ddd/                    # Domain-Driven Design (Evans)
├── event-sourcing-cqrs/    # Greg Young
├── modular-monolith/       # Shopify·Basecamp 스타일
├── solid/                  # Robert Martin 5원칙
├── twelve-factor/          # Heroku 12-Factor App
├── resilience-patterns/    # CB·bulkhead·back-pressure·rate-limit
├── strangler-fig/          # Fowler 레거시 교체
└── team-topologies/        # Skelton·Pais + Conway
```

## 사용

```
/software-framework <상황 설명>
```

메타 라우터가 문제 신호를 매핑해 1~3개 프레임워크를 선택, Skill 툴로 실행·합성.

## 범위 바깥

- 특정 언어·런타임 best practice
- 프레임워크 선택 (React vs Vue, Django vs Rails 등)
- 비즈니스·제품 전략 (→ `think`)
- 협상·커뮤니케이션 (→ `howtotalk`)
