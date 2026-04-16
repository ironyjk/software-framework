---
name: team-topologies
version: "0.1.0"
description: "Team Topologies (Skelton·Pais) — 4팀 유형 + 3 상호작용 모드 + 인지 부하 + Inverse Conway Maneuver. 팀 구조가 아키텍처를 만든다. 팀 확장·플랫폼화·마이크로서비스 조직 설계."
---

# Team Topologies

## 한 줄 요약

**팀 구조가 소프트웨어 구조를 결정한다(Conway's Law).** 그래서 원하는 아키텍처를 만들려면 먼저 팀을 그렇게 조직하라(Inverse Conway Maneuver). 4가지 팀 유형과 3가지 상호작용 모드로 조직을 명시적으로 설계.

## 이론 기원

- **Matthew Skelton & Manuel Pais** — *Team Topologies* (2019). DevOps·애자일 조직 설계 집약.
- 뿌리: Conway's Law (1968), Spotify Model, Dunbar's number.
- 현대 영향: 많은 테크 기업이 플랫폼 엔지니어링 조직 설계에 채택.

## Conway's Law (전제)

> "시스템을 설계하는 조직은 그 조직의 커뮤니케이션 구조를 복제하는 설계를 만든다." — Melvin Conway (1968)

**역명제 (Inverse Conway Maneuver)**: 원하는 아키텍처가 있다면 그 구조로 팀을 먼저 조직하라.

## 4가지 팀 유형

### 1. Stream-Aligned Team
- **가치 스트림 하나에 정렬**된 팀
- End-to-end 책임 (설계·개발·배포·운영)
- 제품의 *대부분*이 여기 속해야 함 (7~8할)
- 예: "결제 팀", "검색 팀", "모바일 iOS 팀"

### 2. Enabling Team
- 한시적 **코칭·멘토링**
- Stream-aligned 팀의 새 기술·실천 습득 도움
- 실행 자체는 하지 않음 (가르침)
- 예: SRE 실천 확산, DDD 도입 지원

### 3. Complicated Subsystem Team
- **깊은 전문지식 필요** 영역
- ML·영상 처리·결제 코어·암호학 등
- Stream-aligned 팀의 부하를 줄임
- 작게 유지 (5~9명)

### 4. Platform Team
- 내부 **플랫폼 제공**
- CI/CD, 관측, 인프라, 공통 라이브러리
- "Platform as a Product" — 내부 사용자 만족이 KPI
- Stream-aligned 팀이 *self-service*로 쓰게

## 3가지 상호작용 모드

### Collaboration
- 두 팀이 밀접 협력, 경계 혼합
- 새 기술·복잡한 문제 탐색 시
- 고비용. 단기에만.

### X-as-a-Service
- 한 팀이 "서비스" 제공, 다른 팀이 "소비"
- 명확한 인터페이스·계약
- 가장 저비용·지속 가능 모드
- Platform Team의 기본 모드

### Facilitating
- Enabling Team이 다른 팀을 돕는 관계
- 일시적, 가르침 중심

팀 간 관계는 항상 이 셋 중 하나로 *명시적*이어야 함. "그냥 가끔 협업" = 불분명 = 마찰.

## 인지 부하 (Cognitive Load)

- 팀이 감당할 수 있는 복잡도는 한계 있음
- 3종 부하: 내재적(도메인 본질) / 외재적(나쁜 도구) / 연관적(학습 활동)
- **외재적 부하를 제거**하고 연관적 부하에 여유 남겨라
- 팀이 담당 영역이 너무 넓으면 → 분할, Platform·Complicated Subsystem 팀이 흡수

## 언제 쓰나

- 조직이 20명·50명·200명 티어 넘을 때 재편
- 모노리스 → 마이크로서비스 이행 시 팀 설계
- 플랫폼 엔지니어링 조직 신설
- 속도 저하·팀 간 대기 시간 증가 진단
- DevOps 성숙 단계 평가

## 실전 적용

### 팀 구성 체크리스트
- [ ] 한 팀당 **2~8명** (Amazon 2-pizza team)
- [ ] 한 팀이 **하나의 가치 스트림** 담당
- [ ] 팀 간 **의존성 수**가 명시적으로 보임 (가급적 3~4개 이하)
- [ ] 각 팀의 **인지 부하**가 감당 가능한가
- [ ] Platform Team이 있으면 "Platform as a Product"로 운영되는가

### Inverse Conway Maneuver 예시

**목표 아키텍처**: 결제·주문·재고가 독립 배포되는 모듈러 모노 + 마이크로

**조직 개편**:
- 결제 Stream-Aligned Team
- 주문 Stream-Aligned Team
- 재고 Stream-Aligned Team
- Platform Team (CI/CD·관측·DB as a Service)
- Payment Core Complicated Subsystem Team (PCI·인증·정산 로직)
- 필요 시 Enabling Team (SRE 확산)

→ 6개월 운영 후 서비스 경계가 *팀 경계를 따라* 자연스럽게 형성됨.

### 팀 API

각 팀이 다음을 공개:
- 소유한 시스템·서비스
- 팀 사용 방법 (상호작용 모드·SLA)
- 로드맵·현재 우선순위
- 연락처·온콜 정보

## 안티패턴

- **기능별 팀 사일로** — "프론트 팀 / 백엔드 팀 / DB 팀"은 stream 방해
- **"Infra 팀에 이슈 던지기"** — X-as-a-Service 아닌 ticket 지옥
- **Platform이 제품이 아닌 경우** — 팀이 쓰기 싫어하면 platform 아님
- **모든 팀이 collaboration 모드** — 협업 피로 누적
- **팀 경계 없이 마이크로서비스** — Conway의 역으로 실패
- **Complicated Subsystem 팀의 블랙박스화** — 소통 안 하면 병목

## 한계

1. **조직 정치 수반** — 팀 재편은 권력·예산 이슈
2. **Dunbar 제약** — 50명+ 조직에서 모든 상호작용 파악은 한계
3. **스타트업 초기엔 과함** — 5~15명 팀이 전체인 단계엔 다 한 팀
4. **고정 분류의 함정** — 팀이 변함에 따라 유형·모드 진화해야
5. **책이 솔루션 제공 아님** — 진단 도구 + 어휘. 적용은 맥락에 맞게.

## 한국 조직 맥락 흔한 실패

- **"플랫폼 본부·개발 본부"식 기능 사일로** — 토스·네이버·카카오가 공통으로 겪은 초기 함정. Stream-aligned 부재 → 모든 프로젝트가 본부 3~4개 핸드오프.
- **"TF(Task Force) 남발"** — 영구적 팀 경계를 못 긋고 TF로 대체 → 소속 모호·책임 분산. Stream-aligned 팀은 *영구 구조*여야 함.
- **창업 멤버·시니어의 광역 소유권** — "A, B, C 모듈 다 아는 사람"이 있으면 역 Conway 적용 시 정치적 저항 발생. 면담·재배치 계획 선행.
- **"플랫폼 실(Platform Team)이 ticket queue"** — Platform as a Product 실패. 셀프서비스 API·문서·SLA 없으면 병목. 토스·쿠팡 사례 다수.
- **팀장·파트장 계층 과다** — 한국 대기업은 TL→파트장→팀장→실장→본부장까지. 의사결정 위임 매트릭스 없으면 모든 결정이 위로.
- **연봉·평가 제도가 기능별 사일로 강화** — "프론트 평가는 프론트 팀에서"가 stream-aligned 형성 방해. HR·평가 제도 동반 설계 필요.
- **조직도 먼저 확정하고 사람 끼워넣기** — 한국식 "인사 발표" 방식은 Team Topologies와 충돌. 팀 설계 → 의향 수렴 → 확정이 순서.

## 이 프레임워크와 함께 쓰는 것들

- `modular-monolith` — 모듈 경계 = 팀 경계 (Conway)
- `ddd` — bounded context = stream-aligned 팀 범위
- `strangler-fig` — 레거시 분할 시 팀 재편과 연동
- `observability` — Platform Team의 대표 서비스

## 이 프레임워크가 *틀렸을 때*

- 10명 이하 조직 — 단일 팀으로 충분
- 극단적 고전문가 집단 (R&D·리서치) — 유동적 조직이 나을 수 있음
- 조직 설계 권한 없는 상태에서 "이렇게 해야" — 정치 없이 추진 불가

## 추가 학습

- Skelton, M. & Pais, M. *Team Topologies.* (원전, 짧고 실용적)
- teamtopologies.com — 공식 리소스·도식
- Forsgren, N. et al. *Accelerate.* — DORA 지표와 팀 설계 연결
- Kim, G. et al. *The Phoenix Project*·*The DevOps Handbook.* — DevOps 조직 이론적 배경
