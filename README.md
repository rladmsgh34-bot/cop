# COP 프로젝트 정보

## 프로젝트 개요
`cop` 저장소는 **n8n + Dify + SonarQube**를 중심으로 구성된 자동화 운영/품질 파이프라인 프로젝트입니다.  
GitHub 이벤트(주로 push)를 기반으로 코드 변경 감지, 품질 분석, 보고/알림, 지식화(RAG)까지 연결하는 것을 목표로 합니다.

---

## 🚀 최신 진행 상황 (2026-04-03)

### 운영 상태
- n8n, Dify, SonarQube 서비스 운영 중
- QA 워크플로우 주요 오류 원인 분석 및 보정 완료
- Sonar Scanner 브릿지(포트 9001) 재구성 및 프로세스 복구

### 최근 반영 사항
- **Self-Healing Loop**
  - Sonar 토큰 이슈 해결
  - 웹훅 E2E 재검증 성공
- **git RAG**
  - GitHub 인증 오류(401) 보정
  - 인증 경로 정리(헤더 기반)
- **QA 워크플로우**
  - “수정할 이슈 없음” 케이스를 실패로 처리하던 로직 개선
  - 메일의 변경사항 링크(API URL → GitHub 웹 URL) 수정
  - Sonar 스캔 경로 복구 및 재검증

### 시크릿/토큰 관리
- Sonar 관련 토큰은 Vault secret으로 관리
- 토큰 메타(활성 토큰, 만료일)도 secret으로 함께 관리

---

## 시스템 아키텍처

```text
GitHub Push
   └─> n8n Webhook
       ├─> Dify (LLM 분석/요약)
       ├─> SonarQube (정적 분석)
       ├─> QA 자동화(리포트/알림)
       └─> KB 적재(RAG)
```

---

## 배포 환경
- 플랫폼: GCP VM (Debian 12)
- 도메인: `uni34.duckdns.org`
- 서비스:
  - n8n: `https://uni34.duckdns.org`
  - Dify: `http://34.64.123.114:8004`
  - SonarQube: `http://34.64.123.114:9000`

---

## 핵심 워크플로우 (n8n)
- `Self-Healing Loop`
- `git RAG`
- `QA`
- `SonarQube Analysis -> Dify / Alerts`

> 워크플로우 상세 ID/노드 변경 내역은 n8n에서 직접 확인하거나 운영 로그 기준으로 추적합니다.

---

## 운영 원칙
- 민감정보(토큰/키)는 코드/문서 하드코딩 금지
- Vault 기반 secret 관리
- 장애 시 원인 → 임시 우회 → 원복 순서로 처리
- 변경 후 E2E 검증(웹훅/실행 이력) 필수

---

## 다음 작업 제안
- QA 워크플로우 Sonar 단계의 예외 분기(권한/프로젝트 미존재) 메시지 고도화
- 워크플로우별 헬스체크/알림 자동화(만료 3일 전 알림 포함)
- README에 운영 점검 체크리스트(일일/주간) 추가

---

_이 문서는 COP 운영 기준으로 계속 갱신됩니다._
