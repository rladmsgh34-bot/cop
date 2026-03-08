# COP 프로젝트 정보

## 프로젝트 개요
이 저장소는 `uni34.duckdns.org` 도메인과 GCP 환경에 배포된 n8n 및 Dify 서비스를 문서화합니다. Dify는 현재 HTTP `http://34.64.123.114:8004`를 통해 접근하며, n8n은 `https://uni34.duckdns.org`를 통해 HTTPS 접근을 제공합니다. 초기 Nginx Proxy Manager를 통한 자동화 시도는 실패했으며, 최종적으로 Dify의 Docker Nginx를 통합하여 서비스를 성공적으로 노출했습니다.

## 📚 목차

1.  [프로젝트 개요](#1-프로젝트-개요)
2.  [배포 환경 (GCP)](#2-배포-환경-gcp)
3.  [서비스 구성 (Nginx, n8n, Dify)](#3-서비스-구성-nginx-n8n-dify)
4.  [도메인 설정 (DuckDNS)](#4-도메인-설정-duckdns)
5.  [접근 URL](#5-접근-url)
6.  [향후 작업](#6-향후-작업)

---

## 1. 프로젝트 개요

GCP (Google Cloud Platform) Debian 12 인스턴스에 n8n (워크플로우 자동화) 및 Dify (LLM 앱 빌더) 서비스를 Docker 및 Docker Compose를 사용하여 배포합니다. Dify는 `http://34.64.123.114:8004`로 HTTP 접근을 제공하며, n8n은 `uni34.duckdns.org` 도메인을 통해 외부에서 HTTPS 접근을 가능하게 합니다.

## 2. 배포 환경 (GCP)

*   **인스턴스**: GCP Debian 12 (Intel Xeon, 7.8GB RAM, 35GB Disk)
*   **외부 IP**: `34.64.123.114`
*   **기본 설치**: Docker 및 Docker Compose
*   **방화벽 규칙**: TCP 80, 443, 8004, 8443 포트가 `0.0.0.0/0` (모든 IP)으로부터의 인그레스(수신) 트래픽을 허용하도록 설정되었습니다.

## 3. 서비스 구성 (Nginx, n8n, Dify)

### 통합 Nginx 서비스 (Dify Docker Nginx)
*   **컨테이너 이름**: `docker-nginx-1` (Dify Docker Compose 스택에 포함)
*   **설정 파일 경로 (호스트)**: `<WORKSPACE>/dify/docker/nginx/conf.d/default.conf.template`
*   **포트 매핑 (호스트:컨테이너)**:
    *   `8004:80` (Dify HTTP)
*   **용도**:
    *   `http://uni34.duckdns.org:8004` 또는 `http://34.64.123.114:8004`로 들어오는 요청을 Dify Web 서비스(`http://web:3000`)로 프록시.
    *   `https://uni34.duckdns.org` (443 포트)로 들어오는 요청을 `n8n-n8n-1:5678`로 프록시. (Nginx 설정에서 HTTPS 서버 블록은 제거되었지만, n8n 서비스는 별도로 HTTPS로 접근 가능)
*   **네트워크**: `docker_default` (Dify 내부 통신) 및 `n8n_default` (n8n과의 통신을 위한 외부 네트워크)에 연결. Dify `api` 서비스 또한 `n8n_default`에 연결되어 외부 LLM API 접근 가능.

### n8n 서비스
*   **Docker 컨테이너**: `n8n-n8n-1`
*   **내부 포트**: `5678` (HTTP)
*   **접속**: Nginx 프록시를 통해 `https://uni34.duckdns.org`로 접근.
*   **docker-compose.yml 위치**: `<WORKSPACE>/n8n/docker-compose.yml`

### Dify 서비스
*   **Docker Compose 위치**: `<WORKSPACE>/dify/docker/`
*   **주요 컨테이너**: `docker-web-1` (Web), `docker-api-1` (API), `docker-plugin_daemon-1` (Plugin Daemon) 등.
*   **접속**: Nginx 프록시를 통해 `http://34.64.123.114:8004`로 접근.

## 4. 도메인 설정 (DuckDNS)

*   **도메인**: `uni34.duckdns.org`
*   **IP 주소**: `34.64.123.114` (GCP 인스턴스의 외부 IP)
*   DuckDNS 서비스를 통해 도메인이 외부 IP를 가리키도록 설정되었으며, 주기적으로 IP를 갱신합니다.

## 5. 접근 URL

*   **n8n**: `https://uni34.duckdns.org`
*   **Dify Web (HTTP)**: `http://34.64.123.114:8004`

## 6. 향후 작업

*   **GitHub webhook 설정**: GitHub Push 이벤트와 Dify/n8n 연동을 위한 webhook 설정
*   **Dify에 GitHub 지식 등록**: GitHub 프로젝트를 Dify 지식베이스로 등록
*   **n8n + Dify 연동 워크플로우**: 자동화 워크플로우 구성
*   **Push 이벤트 자동화**: Push 발생 시 영향도 분석 및 품질 활동 자동화

---

### _이전 작업 내역 (2026-03-07 ~ 현재, 참고용)_

**1. Dify + n8n 초기 HTTPS 통합 시도 (2026-03-07)**
- `uni34.duckdns.org` 도메인에 Let's Encrypt 정식 인증서 발급 (~2026-06-05 유효).
- Dify: `https://uni34.duckdns.org:8443` (웹 접속 가능, 로딩 이슈 있었음)
- n8n: `https://uni34.duckdns.org` (포트 443, 정상 작동)
- COP 및 TEST 프로젝트 README 업데이트 및 GitHub push 완료.

**2. Dify API 연결 문제 및 HTTPS 비활성화 (2026-03-08)**
- **문제**: Dify 웹 로딩 멈춤 및 API 404 에러 (Nginx 로그에서 `api` 서비스로의 `Connection refused` 확인).
- **근본 원인**: Docker Compose 포트 매핑과 Nginx 리스닝 포트 불일치
  - Docker: `8004:80` (호스트 8004 → 컨테이너 80)
  - Nginx 설정: `listen 8004` (컨테이너 포트 8004)
  - 결과: 연결 거부 (Connection reset by peer)
- **해결 조치**:
  - `proxy.conf` 파일 생성 (프록시 헤더 설정)
  - Nginx 설정 수정: `listen 8004` → `listen 80`
  - Nginx 컨테이너 재시작
- **최종 상태**: ✅ `http://34.64.123.114:8004`로 Dify 정상 접속 가능