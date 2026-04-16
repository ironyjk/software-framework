---
name: observability
version: "0.1.0"
description: "Observability — USE(Gregg) + RED(Wilkie) + Four Golden Signals(Google SRE)로 시스템 내부 상태를 외부 신호로 추론. 3 pillars(metrics/logs/traces). 성능·장애 진단의 표준 렌즈."
---

# Observability

## 한 줄 요약

*외부 신호*로 *내부 상태*를 물을 수 있는가? 그게 가능한 만큼이 observability. 성능·장애·용량 질문을 할 때마다 같은 프레임워크로 본다.

## 이론 기원

- **Brendan Gregg** — USE method (2012). 리소스 중심.
- **Tom Wilkie** — RED method (2015). 요청 중심.
- **Google SRE** — Four Golden Signals (*Site Reliability Engineering*, 2016).
- **Charity Majors·Honeycomb** — "high-cardinality observability" 대중화.

## 세 가지 표준 렌즈

### 1. USE (리소스 중심, Gregg)
*모든 리소스에 대해*:
- **U**tilization — 사용 중 시간 비율
- **S**aturation — 대기열·포화 정도
- **E**rrors — 오류 카운트

적용 대상: CPU, 메모리, 디스크, 네트워크, 파일 디스크립터, 스레드·커넥션 풀.

### 2. RED (서비스 중심, Wilkie)
*모든 요청 가능한 서비스에 대해*:
- **R**ate — 초당 요청 수
- **E**rrors — 실패 비율
- **D**uration — 응답 시간 (p50·p95·p99)

마이크로서비스 환경 표준. USE의 "유저 경험" 보완.

### 3. Four Golden Signals (Google SRE)
- **Latency** — 성공·실패 분리
- **Traffic** — 시스템이 받는 부하
- **Errors** — 실패 비율
- **Saturation** — 가장 제약된 리소스의 포화

USE + RED 통합 버전. 유저 향하는 시스템에 먼저 적용.

## Three Pillars of Telemetry

| 유형 | 용도 | 비용 |
|---|---|---|
| **Metrics** | 대시보드·알람·경향 | 낮음 |
| **Logs** | 사건 상세·조사 | 중간 |
| **Traces** | 분산 요청 경로·병목 식별 | 중·고 |

현대: **structured logs + exemplars + traces**를 묶는 OTel(OpenTelemetry) 표준.

## 고 카디널리티와 샘플링

- Cardinality = 태그의 고유 조합 수 (user_id, request_id 포함 시 폭증)
- 전통적 metrics 스토리지(Prometheus 등)는 카디널리티 한계
- High-cardinality observability (Honeycomb·Grafana Loki) → 특정 user·요청의 꼬리 지연 추적 가능
- Sampling: head-based vs tail-based. 에러·느린 요청 우선 보존.

## 핵심 지표 예시

| 지표 | 좋은 값 | 경고 |
|---|---|---|
| API p99 latency | <서비스 SLO | SLO 근접 |
| Error rate | <0.1% | 1%+ |
| CPU util | <70% | >85% |
| DB conn pool sat | <50% | >80% |
| GC pause | <100ms | >500ms |

## 언제 쓰나

- 성능 저하 원인 분석 시작점
- 용량 계획 (현재 포화도 → 성장 대비)
- SLO 정의·준수 모니터링
- 장애 진단 첫 렌즈
- 배포 영향 관찰 (카나리·블루그린)

## 실전 적용

### 사고 진단 순서
1. **Golden Signals 대시보드** — 어디가 빨간가?
2. **RED**로 영향받은 서비스 식별 — rate·error·duration 어떤 게?
3. **USE**로 리소스 병목 — CPU·메모리·풀·디스크?
4. **Traces**로 요청 경로의 특정 구간 지연 찾기
5. **Logs**로 원인 스택·에러 메시지 확인

### SLI/SLO 설계
- SLI = 측정 가능한 지표 (p99 latency, error rate)
- SLO = 목표 (지난 30일 p99 < 300ms, 99.9%)
- Error budget = (1 - SLO) 내에서 실험 가능
- Alerting은 SLO burn rate 기반 (Google SRE Workbook)

## 안티패턴

- **"모든 걸 저장"** — 비용 폭발, signal-to-noise 하락
- **Alert fatigue** — 문턱치 알람 남발 → 진짜 신호 놓침
- **CPU%만 보기** — saturation·errors 없이 단일 지표 오판
- **평균으로 latency 판단** — p50은 거짓말. p95/p99 필수
- **카디널리티 폭주** — user_id를 태그로 (→ 인덱스 폭발)

## 한계

1. **계측 비용** — 모든 신호를 다 수집·저장·질의 못함. 우선순위 필요.
2. **알려진 미지만 봄** — 이미 달아둔 신호만 볼 수 있음. unknown unknowns는 high-cardinality 도구 필요.
3. **설정 시차** — 장애 터진 후 필요한 신호 없음을 깨달음이 흔함
4. **상관관계 ≠ 인과관계** — 관측으로 *어디*는 알지만 *왜*는 `scientific-debugging`

## 이 프레임워크가 *틀렸을 때*

- 근본원인 분석 → `scientific-debugging`
- 장애 격리·복원력 설계 → `resilience-patterns`
- 코드 레벨 성능 프로파일링 → 언어별 프로파일러 (관측 바깥)

## 한국 현장 도구·맥락

- **상용 APM**: Datadog·New Relic·Dynatrace (토스·쿠팡·배민 주류)
- **오픈소스**: Prometheus + Grafana + Loki + Tempo, Elastic Stack
- **국산/국내 호스팅**: Pinpoint(네이버 오픈소스, Java APM), Scouter(LG CNS 오픈소스)
- **한국 클라우드**: NHN Cloud Log & Crash, Naver Cloud Monitoring, KT Cloud 모니터링
- **금융권 망분리 환경**: 외부 SaaS APM 반입 제한 → 내부 자체 구축 많음(ELK + Grafana 조합). OpenTelemetry Collector로 메트릭만 반출 패턴.
- **금감원·전자금융업 보고**: 전자금융감독규정상 장애 보고 의무 존재 (기존 "10분 이상 중단·지연 시 지체 없이 보고", 2025년 개정으로 가입자 1만 명 미만 서비스는 30분 기준으로 완화). 구체 기준은 최신 규정·시행세칙 확인 필요. MTTR·Error budget 설계 시 이 규제를 상한으로 반영.
- **토스·카카오페이 관측 공개 사례**: 블로그·Tech 세션에서 RED+USE+Golden Signals 혼용. Pinpoint→Datadog/Prometheus 마이그레이션 사례 다수.

## 추가 학습

- Gregg, B. *Systems Performance.* (USE method)
- Google SRE. *Site Reliability Engineering*, *SRE Workbook* (무료 공개, sre.google).
- Majors, C., Fong-Jones, L., Miranda, G. *Observability Engineering.* O'Reilly, 2022.
- OpenTelemetry (opentelemetry.io)
- Pinpoint (github.com/pinpoint-apm/pinpoint) — 네이버 오픈소스
