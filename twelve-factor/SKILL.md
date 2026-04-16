---
name: twelve-factor
version: "0.1.0"
description: "Twelve-Factor App (Adam Wiggins, Heroku) — 클라우드 네이티브 앱의 12원칙. 배포·운영·확장의 기본 위생. 설정·의존성·포트·로그·프로세스 12개 축으로 체크."
---

# Twelve-Factor App

## 한 줄 요약

**클라우드에 바로 올릴 수 있는 앱의 기본 위생 12가지**. 대부분은 이제 상식처럼 느껴지지만, 위반 사례는 2026년에도 흔하다.

## 이론 기원

- **Adam Wiggins** 외 Heroku 엔지니어들 (2011) — `12factor.net`.
- 수천 개 SaaS 앱 운영 경험의 집약.
- Kubernetes·Docker 생태계의 기저 전제로 자리잡음.

## 12개 원칙

### I. Codebase
**한 앱 = 하나의 코드베이스, 여러 배포**
- 한 Git 레포 = 한 앱
- 레포 여럿이 같은 앱 = 위반 (공유 라이브러리로 분리해야)
- 모노레포 내 여러 앱은 OK

### II. Dependencies
**의존성을 명시적으로 선언, 격리**
- `package.json`, `requirements.txt`, `go.mod`, `Gemfile`
- 시스템 패키지 암묵 의존 금지
- lock file 커밋

### III. Config
**환경별로 달라지는 모든 것을 환경변수로**
- DB URL, API 키, feature flag
- 코드에 박지 말 것
- 12-Factor 판별 테스트: "공개해도 되는가?" — 못 하면 config에 있어야

### IV. Backing Services
**백엔드 서비스를 교체 가능한 자원으로 취급**
- DB, 캐시, MQ는 URL 하나로 참조
- 로컬 Postgres ↔ RDS 교체 코드 변경 없이

### V. Build, Release, Run
**3단계 엄격 분리**
- Build: 소스 → 실행 번들
- Release: 번들 + config → 릴리스
- Run: 릴리스 실행
- 프로덕션에서 코드 수정 금지

### VI. Processes
**앱을 무상태 프로세스로**
- 메모리·로컬 디스크를 영속 저장소로 쓰지 말 것
- 세션은 Redis 같은 backing service로
- Scaling = 프로세스 추가

### VII. Port Binding
**서비스를 포트 바인딩으로 노출**
- 웹 서버(nginx·Apache)에 의존 않고 self-contained
- Node/Go/Flask가 포트 직접 열고 HTTP 응대
- 라우팅·SSL은 상위 레이어(로드밸런서·ingress)

### VIII. Concurrency
**프로세스 모델로 확장**
- 수직 확장(큰 머신) 대신 수평(프로세스 추가)
- Unix 프로세스 모델: web / worker / scheduler 등 프로세스 타입
- OS가 프로세스 관리(systemd, K8s)

### IX. Disposability
**빠른 시작, graceful 종료**
- 몇 초 내 시작
- SIGTERM 받으면 현재 작업 정리 후 종료
- 크래시 안전: 재시작으로 복구

### X. Dev/Prod Parity
**개발·스테이징·프로덕션을 최대한 비슷하게**
- 시간 차: 배포 빈도 높게 (며칠 → 시간)
- 인력 차: 개발자가 배포도
- 도구 차: dev와 prod가 같은 DB·MQ

### XI. Logs
**로그를 이벤트 스트림으로**
- 앱은 stdout/stderr로 흘려보냄
- 수집·저장·분석은 환경 책임 (Stackdriver, ELK, Loki)
- 앱이 로그 파일 관리 X

### XII. Admin Processes
**관리 작업을 일회성 프로세스로**
- 마이그레이션, 일회 스크립트도 같은 코드베이스·같은 릴리스로 실행
- 프로덕션 REPL 접속도 동일 환경

## 언제 쓰나

- 클라우드 네이티브 앱 설계·리팩터
- 레거시 클라우드 마이그레이션 체크리스트
- 신입 온보딩 공통 언어
- 배포 자동화 전 기반 점검
- Kubernetes·ECS 전환 전제 조건

## 실전 적용

### 체크리스트로
각 factor를 5분씩 점검:
- [ ] Config가 env var로?
- [ ] 로그가 stdout?
- [ ] 빌드·릴리스·런 분리?
- [ ] 프로세스 무상태?
- [ ] 시작·종료 시간?

위반 한두 개 발견 = 흔함. 전 항목 즉시 고치지 말고 *운영 고통과 연결된 것부터*.

### 모던 확장 ("Beyond 12 Factor", Kevin Hoffman)
- API First
- Telemetry
- Authentication & Authorization
- Self-service
- 더 분화된 backing service 이해

## 안티패턴

- **Config를 YAML 파일로 커밋** — 환경별 다르면 위반
- **로컬 파일에 세션·캐시 저장** — 프로세스 재시작 시 소실
- **로그 파일 로테이션을 앱이** — 환경에 맡길 일
- **서비스가 OS 의존 바이너리** — 컨테이너·이식성 깨짐
- **"dev DB는 SQLite, prod은 Postgres"** — Parity 위반

## 한계

1. **웹/API 중심** — 임베디드·ML 학습 잡·데스크톱 앱엔 일부만 적용
2. **상태성 있는 서비스엔 한계** — Kafka·DB 운영 자체엔 다른 원칙
3. **세부 구현은 생태계 발전으로 대체됨** — K8s·서비스 메시·OTel
4. **문화적 변화 동반 필요** — DevOps·CI/CD 없이 12-factor만 적용 불가

## 이 프레임워크와 함께 쓰는 것들

- `observability` — XI(로그) 현대판
- `resilience-patterns` — IX(disposability)·VIII(concurrency) 보완
- `modular-monolith` — 내부 구조 원칙
- *조합 효과*: 12-factor 준수 = 후속 리질리언스·관측 도입 비용 급감

## 이 프레임워크가 *틀렸을 때*

- 단일 사용자 CLI·데스크톱
- 임베디드·IoT 제약 환경
- 데이터플레인(DB·MQ 자체) 운영

## 추가 학습

- **원전**: 12factor.net (무료, 짧음, 전문 추천)
- Hoffman, K. *Beyond the Twelve-Factor App.* (2016)
- CNCF 문서 및 Kubernetes 공식 튜토리얼
