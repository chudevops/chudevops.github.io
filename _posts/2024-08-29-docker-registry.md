---
layout: post
title: "Docker Registry 설정 및 관리 자동화"
date: 2024-08-29 08:00:00 +0900
categories: [Docker, Registry]
tags: [docker, registry, automation]
toc: true
reading_time: 10분
description: "Docker Registry 서버 설치 부터 이미지 Create 및 Run 작업을 자동화하는 방법을 설명합니다. Docker Hub에서 이미지를 Pull 하고 컨테이너로 실행하는 스크립트를 다룹니다."
---

## 소개

Docker Registry는 Docker 이미지를 저장하고 관리하는 서버로 여러 서버 간 이미지 공유와 프로젝트별 이미지 버전 관리를 지원합니다.  
Docker Hub에서 이미지를 Pull 하고 Docker Registry 서버를 실행하는 방법을 설명합니다.  
이미지 Create, Pull, Run 작업 스크립트를 통해 자동화하는 방법을 다룹니다.

### 목표

- Docker Registry 설치 및 관리 간소화
- Docker 이미지의 Create, Pull, Run 작업 자동화
- 스크립트를 통한 효율적인 배포 및 유지보수

### 주요 특징

- **이미지 생성**: 스크립트를 사용하여 Docker Registry 이미지를 생성하고 태깅
- **이미지 Pull**:  Docker Hub에서 이미지를 풀하는 스크립트 제공
- **컨테이너 실행**: Docker Registry 컨테이너 설정 및 실행 자동화
- **자동화**: 스크립트를 통해 반복 작업을 줄이고 효율적인 운영

## 목차

1. [기술 스택](#기술-스택)
2. [프로젝트 구조](#프로젝트-구조)
3. [아키텍처](#아키텍처)
4. [주요 구성 요소](#주요-구성-요소)
5. [기술적 과제](#기술적-과제)
6. [사용 방법](#사용-방법)
7. [결론](#결론)
8. [향후 계획](#향후-계획)

## 기술 스택

- Base Image: Docker Registry
- Automation Scripts: Shell Scripts
- Container: Docker

## 프로젝트 구조

> **소스 코드**는 [GitHub 레포지토리](https://github.com/chudevops/docker-build/tree/master/registry)에서 확인할 수 있습니다.

```plaintext
registry/
├── build.sh
├── pull_image.sh
└── run.sh
```

## 아키텍처

![docker Registry Architecture](/assets/img/2024-08-29-docker-registry/docker_registry_architecture.png)

### Docker Registry 구성 요소

- **Registry**: Docker 이미지를 저장하고 관리하는 서비스
- **Docker Daemon**: Docker 컨테이너를 관리하고 실행하는 프로세스
- **Blob**: Docker 이미지의 각 레이어를 저장하는 데이터 단위로 여러 Blob을 조합해 이미지를 구성
- **Manifest**: Blob 데이터를 참조하는 메타데이터로 이미지에 어떤 Blob이 포함되어 있는지 설명
- **Docker Client**: 이미지를 build, push, pull, run하는 클라이언트로 Registry와 상호작용
- **Docker Host**: Docker 이미지를 빌드하고 Registry로 이미지를 push하거나 pull한 이미지를 실행하는 서버

### Docker Registry 동작 과정

- **Docker Build**
   1. Docker Client가 Docker Daemon에게 빌드 명령을 전달
   2. Docker Daemon이 Dockerfile을 읽고 필요한 이미지 레이어(Blob)를 생성
   3. Blob 데이터와 관련된 메타데이터(Manifest)가 Docker Host의 스토리지에 저장

- **Docker Push**
   1. Docker Client가 Docker Daemon에게 이미지를 Registry로 전송하도록 요청
   2. Docker Daemon이 Registry Server에 연결하여 Manifest와 Blob 데이터를 전송
   3. Registry Server가 해당 데이터를 스토리지(Manifest Storage, Blob Storage)에 저장
   4. Registry Server가 성공적으로 데이터를 저장했음을 Docker Client에 알림

- **Docker Pull**
   1. Docker Client가 Docker Daemon에게 이미지를 요청
   2. Docker Daemon이 Registry Server에 이미지를 요청
   3. Registry Server가 Manifest와 Blob 데이터를 Docker Host의 Docker Daemon에 전달
   4. Docker Daemon이 해당 데이터를 받아 스토리지에 저장
   5. Docker Daemon가 성공적으로 이미지를 저장했음을 Docker Client에 알림

- **Docker Run**
   1. Docker Client가 Docker Daemon에게 컨테이너 실행을 요청
   2. Docker Daemon이 저장된 Blob과 Manifest 데이터를 사용하여 컨테이너를 생성
   3. 컨테이너가 생성되고 애플리케이션 실행이 완료되었음을 Docker Client에 알림

## 주요 구성 요소

### 1. **이미지 생성 스크립트**
Docker Hub에서 Docker Registry 이미지를 다운로드하고 로컬 또는 지정된 레지스트리에 태깅하는 작업을 자동으로 처리합니다.

### 2. **이미지 Pull 스크립트**
Docker Hub에서 최신 Docker Registry 이미지를 가져옵니다.

### 3. **Registry 실행 스크립트**
Docker Registry 컨테이너를 백그라운드에서 실행하며 포트와 볼륨 설정이 포함됩니다.

## 기술적 과제

1. **이미지 다운로드 및 태깅 자동화**
   - 문제: Docker Registry 이미지를 자동으로 다운로드하고 사용자 지정 레지스트리로 태깅해야 함
   - 해결: 이미지 다운로드 및 태깅 작업을 자동화하여 수동 작업을 줄임

2. **이미지 Pull 자동화**
   - 문제: Docker Hub에서 최신 이미지를 가져오는 작업을 효율적으로 처리해야 함
   - 해결: 이미지를 자동으로 Pull 하여 Docker Host에 저장하는 프로세스를 도입함

3. **컨테이너 설정 및 실행 자동화**
   - 문제: Docker Registry 컨테이너의 설정과 실행을 수동으로 처리하는 것은 번거로움
   - 해결: 컨테이너 설정과 실행을 자동화하여 운영 효율성을 높임

## 사용 방법

1. 이미지 생성 및 태깅
```bash
cd registry
./build.sh [custom_registry_url]
```

2. 이미지 다운로드
```bash
./pull_image.sh
```

3. Registry Run
```bash
./run.sh
```

## 결론

- **효율적인 Docker Registry 관리**: 설치 및 설정 과정을 자동화하여 운영을 간소화
- **편리한 이미지 관리**: 이미지를 쉽게 생성, 태깅하고, 레지스트리에 등록 필요한 이미지를 빠르게 가져와 실행
- **재사용 가능한 자동화**: 스크립트는 다양한 환경에서 재사용 가능하고 배포와 유지보수 작업을 간편하게 구성

## 향후 계획

- **성능 최적화**: 이미지 관리와 컨테이너 실행 성능을 개선하여 운영 효율성 높임
- **자동화 범위 확대**: Docker Registry 관리와 보안 설정을 자동화하는 추가 도구 개발 예정
- **보안 강화**: SSL 인증서 기반 보안 통신과 접근 제어를 강화

## 참고 자료

- [Docker 공식 문서](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Registry 공식 문서](https://hub.docker.com/_/registry)
