# COP 프로젝트 정보

## 프로젝트 개요
이 저장소는 `uni34.duckdns.org` 도메인과 GCP 환경에 구축된 **n8n + Dify 기반 GitHub 영향도 및 품질 자동화 시스템** 프로젝트입니다. 
GitHub에 코드가 Push될 때마다 AI가 변경 사항을 분석하여 시스템 영향도를 평가하고, 코드 품질 활동을 자동화하는 것을 목표로 합니다.

## 🚀 최신 진행 상황 (2026-03-10)

*   **서버 설정 완료**: Google Cloud Platform(GCP) Debian 12 환경에 Docker 기반 인프라 구축 완료.
*   **핵심 시스템 배포**: n8n, Dify 서비스 배포 및 안정화 완료.
*   **초기 모델 개발 완료**: n8n + Dify를 활용한 GitHub 영향도 및 품질 자동화 시스템의 초기 모델 개발 완료.
*   **SonarQube 작업 진행 중**: 정적 코드 분석 및 품질 게이트 자동 연동을 위한 SonarQube 구축 및 n8n 연동 작업 진행 중.

## 📚 목차

1.  [시스템 아키텍처](#1-시스템-아키텍처)
2.  [배포 환경 (GCP)](#2-배포-환경-gcp)
3.  [서비스 구성 (Nginx, n8n, Dify, SonarQube)](#3-서비스-구성-nginx-n8n-dify-sonarqube)
4.  [접근 URL](#4-접근-url)
5.  [향후 작업](#5-향후-작업)

---

## 1. 시스템 아키텍처

```
GitHub Push ──→ Webhook ──→ n8n Workflow ──┬──→ Dify (AI Impact Analysis)
                                           └──→ SonarQube (Static Analysis) ──→ Feedback (GitHub/Message)
```

## 2. 배포 환경 (GCP)

*   **인스턴스**: GCP Debian 12 (Intel Xeon, 7.8GB RAM, 35GB Disk)
*   **외부 IP**: `34.64.123.114`
*   **도메인**: `uni34.duckdns.org` (DuckDNS)
*   **방화벽 규칙**: TCP 80, 443, 8004, 9000 포트 허용

## 3. 서비스 구성

### 핵심 자동화 엔진
*   **n8n**: 워크플로우 자동화 트리거 및 데이터 파이프라인 관리.
*   **Dify**: LLM 기반 코드 변경 영향도 분석 및 의사결정 지원 AI.
*   **SonarQube**: 정적 코드 분석을 통한 코드 품질 지표 제공 (진행 중).

### 인프라 서비스
*   **Nginx**: 리버스 프록시 및 SSL 인증서(Let's Encrypt) 관리.
*   **Docker & Docker Compose**: 모든 서비스의 컨테이너화 및 오케스트레이션.

## 4. 접근 URL

*   **n8n (Automation)**: `https://uni34.duckdns.org` ✅
*   **Dify (AI Platform)**: `http://34.64.123.114:8004` ✅
*   **SonarQube (Quality)**: `http://34.64.123.114:9000` 🔄 (설정 중)

## 5. 향후 작업

*   **SonarQube 상세 분석 로직 연동**: n8n 워크플로우에서 SonarQube 분석 결과를 수신하여 AI 분석에 반영.
*   **영향도 분석 AI 고도화**: Dify 지식 베이스 최적화 및 프롬프트 개선.
*   **품질 활동 리포트 자동화**: Slack/Telegram 등으로 상세 품질 리포트 자동 발송.

---

> 이 저장소는 AI 비서 **또치**가 프로젝트의 실시간 진행 상황에 맞춰 자동으로 관리합니다.
