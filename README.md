# COP 프로젝트 정보

## 프로젝트 개요
이 저장소는 `uni34.duckdns.org` 도메인과 GCP 환경에 구축된 **n8n + Dify 기반 GitHub 영향도 및 품질 자동화 시스템** 프로젝트입니다.
GitHub에 코드가 Push될 때마다 AI가 변경 사항을 분석하여 시스템 영향도를 평가하고, 코드 품질 활동을 자동화하는 것을 목표로 합니다.

### 통합 목표
- Dify 및 n8n 서비스를 통한 자동화와 품질 데이터 관리.
- GitHub Push 이벤트 기반 상호 시스템 연동.
- 정적 코드 분석 도구(SonarQube)를 활용한 품질 개선.

---

## 🚀 최신 진행 상황 (2026-03-15)

- **서버 설정 완료**: GCP Debian 12 환경의 초기 구성 완료.
- **핵심 시스템 안정화**: n8n 및 Dify HTTP/HTTPS 서비스 구축 성공.
- **초기 모델 성공**: Git webhook 입력-분석-결과 자동 보고 흐름 시험 작동.
- **SonarQube 추가 작업**: 분석 결과를 n8n 및 Telegram와 통합 (80% 진행).

---

## 📚 목차

1. [시스템 아키텍처](#시스템-아키텍처)
2. [배포 환경](#배포-환경)
3. [서비스 구성](#서비스-구성)
4. [도메인 설정](#도메인-설정)
5. [관리 대상](#관리-대상)
6. [향후 작업](#향후-작업)

---

## 시스템 아키텍처

```
GitHub Push ──→ Webhook ──→ n8n Workflow ──┬──→ Dify (LLM 분석 AI)
                                           └──→ SonarQube (Static Analysis) ───→ Feedback

```

---

## 배포 환경

- **플랫폼**: GCP (Google Cloud Platform)
- **운영체제**: Debian 12
- **도메인 및 IP 정보:**
  - n8n / HTTPS 서버: [https://uni34.duckdns.org](https://uni34.duckdns.org)
  - AI (HTTP): [34.64.123.114:8004](http://34.64.123.114:8004)
  - SonarQube(품질 데쉬):  접근 `http 결과`
...
json
...