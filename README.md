# COP Project Setup Guide

이 저장소는 `uni34.duckdns.org` 도메인을 사용하여 n8n 및 Dify 서비스를 GCP 환경에 배포하고 HTTPS로 안전하게 노출하는 과정을 문서화합니다.

## 📚 목차

1.  [프로젝트 개요](#1-프로젝트-개요)
2.  [배포 환경 (GCP)](#2-배포-환경-gcp)
3.  [서비스 구성 (Nginx, n8n, Dify)](#3-서비스-구성-nginx-n8n-dify)
4.  [SSL/HTTPS 설정 (자체 서명 인증서)](#4-sslhttps-설정-자체-서명-인증서)
5.  [도메인 설정 (DuckDNS)](#5-도메인-설정-duckdns)
6.  [접근 URL](#6-접근-url)
7.  [향후 작업](#7-향후-작업)

---

## 1. 프로젝트 개요

GCP (Google Cloud Platform) Debian 12 인스턴스에 n8n (워크플로우 자동화) 및 Dify (LLM 앱 빌더) 서비스를 Docker 및 Docker Compose를 사용하여 배포하고, `uni34.duckdns.org` 도메인으로 외부에서 HTTPS 접근을 가능하게 하는 것이 목표입니다. 초기 Nginx Proxy Manager를 통한 자동화 시도는 실패했으며, 최종적으로 수동 Nginx 설정을 통해 성공적으로 서비스를 노출했습니다.

## 2. 배포 환경 (GCP)

*   **인스턴스**: GCP Debian 12 (Intel Xeon, 7.8GB RAM, 35GB Disk)
*   **외부 IP**: `34.64.123.114`
*   **기본 설치**: Docker 및 Docker Compose
*   **방화벽 규칙**: TCP 80, 443, 8000, 8443 포트가 `0.0.0.0/0` (모든 IP)으로부터의 인그레스(수신) 트래픽을 허용하도록 설정되었습니다.

## 3. 서비스 구성 (Nginx, n8n, Dify)

### n8n 서비스
*   Docker 컨테이너로 배포
*   내부 포트: `5678`

### Dify 서비스
*   Docker Compose로 배포 (여러 컨테이너 포함: `docker-web-1`, `docker-api-1`, `docker-plugin_daemon-1` 등)
*   내부 포트: `3000` (Web), `5001` (API), `5002` (Plugin Daemon)

### Nginx for n8n (`manual_nginx_n8n`)
*   **컨테이너 이름**: `manual_nginx_n8n`
*   **설정 파일**: `~/nginx_manual/conf.d/default.conf`
*   **Docker Compose 파일**: `~/nginx_manual/docker-compose.yml`
*   **네트워크 모드**: `host` (호스트 네트워크 직접 사용)
*   **포트**: 호스트의 80번, 443번 포트 사용
*   **프록시 대상**: `https://uni34.duckdns.org` → `http://localhost:5678` (n8n)
*   **주요 설정**:
    ```nginx
    server {
        listen 80;
        listen [::]:80;
        server_name uni34.duckdns.org;
        return 301 https://$server_name$request_uri; # HTTP to HTTPS 리다이렉트
    }
    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        server_name uni34.duckdns.org;
        # SSL 인증서 경로 설정
        ssl_certificate /etc/letsencrypt/uni34_cert.pem;
        ssl_certificate_key /etc/letsencrypt/uni34_key.pem;
        location / {
            proxy_pass http://localhost:5678; # n8n 컨테이너로 프록시
            # ... (기타 프록시 헤더) ...
        }
    }
    ```

### Nginx for Dify (`manual_nginx_dify`)
*   **컨테이너 이름**: `manual_nginx_dify`
*   **설정 파일**: `~/nginx_dify/conf.d/default.conf`
*   **Docker Compose 파일**: `~/nginx_dify/docker-compose.yml`
*   **네트워크 모드**: `docker_default` (Dify와 동일한 Docker 네트워크 사용)
*   **포트**: 호스트의 8000번, 8443번 포트 사용
*   **프록시 대상**: `https://dify.uni34.duckdns.org:8443` → `http://docker-web-1:3000`, `http://docker-api-1:5001`, `http://docker-plugin_daemon-1:5002`
*   **주요 설정**:
    ```nginx
    server {
        listen 8000;
        listen [::]:8000;
        server_name dify.uni34.duckdns.org;
        return 301 https://$server_name:8443$request_uri; # HTTP to HTTPS 리다이렉트
    }
    server {
        listen 8443 ssl;
        listen [::]:8443 ssl;
        server_name dify.uni34.duckdns.org;
        # SSL 인증서 경로 설정
        ssl_certificate /etc/letsencrypt/dify_cert.pem;
        ssl_certificate_key /etc/letsencrypt/dify_key.pem;
        location / {
            proxy_pass http://docker-web-1:3000; # Dify Web 컨테이너로 프록시
            # ... (기타 프록시 헤더) ...
        }
        location /api {
            proxy_pass http://docker-api-1:5001; # Dify API 컨테이너로 프록시
            # ...
        }
        location /e/ {
            proxy_pass http://docker-plugin_daemon-1:5002; # Dify Plugin Daemon으로 프록시
            # ...
        }
        # ... (기타 Dify 관련 프록시 설정) ...
    }
    ```

## 4. SSL/HTTPS 설정 (자체 서명 인증서)

초기 Let's Encrypt 자동 발급 시도 실패 후, 자체 서명(Self-Signed) 인증서를 OpenSSL로 생성하여 Nginx에 적용했습니다.

*   **생성 명령어 예시**:
    ```bash
    openssl req -x509 -newkey rsa:2048 -keyout uni34_key.pem -out uni34_cert.pem -days 365 -nodes -subj "/CN=uni34.duckdns.org"
    ```
*   **경고**: 자체 서명 인증서는 브라우저에서 `NET::ERR_CERT_AUTHORITY_INVALID`와 같은 보안 경고를 표시합니다. 이는 예상된 동작이며, **"안전한 페이지로 들어가기"** 또는 **"계속 진행"**을 통해 접속할 수 있습니다.

## 5. 도메인 설정 (DuckDNS)

*   **도메인**: `uni34.duckdns.org` 및 `dify.uni34.duckdns.org`
*   **IP 주소**: `34.64.123.114` (GCP 인스턴스의 외부 IP)
*   DuckDNS 서비스를 통해 도메인이 외부 IP를 가리키도록 설정되었으며, 주기적으로 IP를 갱신합니다.

## 6. 접근 URL

*   **n8n**: `https://uni34.duckdns.org`
*   **Dify**: `https://dify.uni34.duckdns.org:8443`

## 7. 향후 작업

*   **Let's Encrypt 정식 인증서 적용**: 브라우저 보안 경고를 제거하기 위해 Let's Encrypt를 통한 정식 SSL 인증서 발급 및 적용을 진행할 수 있습니다.
*   **인증서 자동 갱신**: Certbot을 사용하여 Let's Encrypt 인증서의 자동 갱신을 설정합니다.
*   **Dify Nginx 포트 정리**: 현재 Dify Nginx는 포트 8000/8443을 사용하지만, 표준 HTTPS 포트 443을 사용하도록 재설정하고, n8n과의 충돌을 피하기 위한 추가적인 조치 (예: 다른 서브도메인에 대한 Nginx 인스턴스 분리 등)를 고려할 수 있습니다.
