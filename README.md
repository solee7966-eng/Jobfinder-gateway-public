#  JobFinder — API Gateway

> JobFinder 플랫폼의 단일 진입점(Single Entry Point) 서비스입니다.  
> 전체 프로젝트 소개는 → **[Main Service README](https://github.com/solee7966-eng/Jobfinder-main-public)** 를 참고하세요.

[![Java](https://img.shields.io/badge/Java-17-007396?style=flat-square&logo=java)](https://www.java.com)
[![Spring Cloud Gateway](https://img.shields.io/badge/Spring_Cloud-Gateway-6DB33F?style=flat-square&logo=springboot)](https://spring.io/projects/spring-cloud-gateway)
[![Eureka](https://img.shields.io/badge/Eureka-Client-6DB33F?style=flat-square&logo=spring)](https://spring.io/projects/spring-cloud-netflix)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)](https://www.docker.com)

**포트** | `8000` &nbsp;·&nbsp; **역할** | 클라이언트 요청을 각 마이크로서비스로 라우팅

---

##  이 서비스의 역할

API Gateway는 클라이언트의 **모든 요청이 거치는 단일 진입점**입니다.  
Discovery Service(Eureka)와 연동하여 서비스 이름 기반으로 동적 라우팅을 처리하며,  
각 마이크로서비스의 엔드포인트를 클라이언트에 직접 노출하지 않고 Gateway를 통해 관리합니다.

```
Client (Browser)
    │
    ▼
┌─────────────────────────────┐
│     API Gateway (8000)      │  ⬅️ 현재 레포
│  - Single Entry Point       │
│  - Service Routing          │
│  - Discovery Lookup         │
└─────────────┬───────────────┘
              │  Discovery Service(8761) 조회 후 라우팅
    ┌─────────┴──────────┐
    ▼                    ▼
Main Service (8001)  Board Service (8002)
```

---

##  주요 역할

- **라우팅**: 요청 경로 기반으로 Main Service / Board Service로 동적 라우팅
- **Discovery 연동**: Eureka Client로 등록된 서비스 목록을 조회해 서비스 이름 기반 라우팅 처리
- **단일 진입점**: 클라이언트는 Gateway 주소(`:8000`)만 알면 되며, 내부 서비스 포트를 노출하지 않음

---

##  라우팅 구조

```yaml
# application.yml 라우팅 예시
spring:
  cloud:
    gateway:
      routes:
        - id: main-service
          uri: lb://MAIN-SERVICE        # Eureka 서비스명 기반 로드밸런싱
          predicates:
            - Path=/company/**, /user/**

        - id: board-service
          uri: lb://BOARD-SERVICE
          predicates:
            - Path=/board/**
```

> `lb://` 접두사를 통해 Eureka에 등록된 서비스 이름으로 동적 라우팅합니다.  
> 서비스 인스턴스 주소가 바뀌어도 Gateway 설정 변경 없이 자동 반영됩니다.

---

##  기술 스택

| 분류 | 기술 |
|------|------|
| **Backend** | Java, Spring Cloud Gateway |
| **서비스 디스커버리** | Spring Cloud Netflix Eureka Client |
| **인프라** | AWS EC2, Docker, Docker Compose |
| **CI/CD** | Jenkins |

---

##  실행 방법

> ⚠️ Gateway 실행 전 **Discovery Service(8761)** 가 먼저 실행되어 있어야 합니다.  
> Gateway → Main/Board Service 순서로 실행하세요.

```bash
# 레포지토리 클론
git clone https://github.com/solee7966-eng/Jobfinder-gateway-public.git
cd Jobfinder-gateway-public

# 환경변수 설정 (application.yml)
# Eureka 서버 주소 설정 필요
# eureka.client.service-url.defaultZone=http://localhost:8761/eureka/

# 빌드 및 실행
./gradlew build
java -jar build/libs/*.jar

# Docker로 실행하는 경우
docker-compose up --build
```

---

## 📁 관련 레포지토리

| 레포 | 포트 | 역할 |
|------|------|------|
| [Jobfinder-main-public](https://github.com/solee7966-eng/Jobfinder-main-public) | 8001 | 메인 서비스 (회원·공고·결제) — **전체 소개 여기** |
| [Jobfinder-board-public](https://github.com/solee7966-eng/Jobfinder-board-public) | 8002 | 커뮤니티 게시판 서비스 |
| [Jobfinder-gateway-public](https://github.com/solee7966-eng/Jobfinder-gateway-public) | 8000 | ⬅️ 현재 레포 (API Gateway) |
| [Jobfinder-discovery-public](https://github.com/solee7966-eng/Jobfinder-discovery-public) | 8761 | Eureka Discovery 서비스 |
