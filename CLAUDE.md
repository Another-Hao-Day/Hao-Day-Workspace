# Hao-Day 통합 워크스페이스

> 이 워크스페이스에서 Claude Code를 실행하면 Backend, Infra, App 모든 레포에 접근할 수 있습니다.

**마지막 업데이트**: 2026-01-23

---

## 프로젝트 개요

**Hao-Day (오늘도 하오!)** - 듀오링고 스타일의 중국어 학습 플랫폼

### 레포지토리 구조

| 레포 | 경로 | 기술 스택 | 설명 |
|------|------|-----------|------|
| **Backend** | `../Hao-Day-Backend` | Java 17, Spring Boot 3.2.2, MySQL 8, Redis 7 | REST API 서버 |
| **Infra** | `../Hao-Day-Infra` | Docker, Nginx, GitHub Actions | 인프라 설정 |
| **App** | `../Hao-Day-App` | Flutter 3.38, Dart 3.10, Provider | 모바일 앱 |

---

## 용어 정의 (자연어 컨텍스트 전환)

Claude에게 다음 용어로 레포를 지정할 수 있습니다:

| 용어 | 대상 레포 | 경로 |
|------|-----------|------|
| 앱, 프론트, 클라이언트, Flutter, Dart, 모바일 | Hao-Day-App | `../Hao-Day-App` |
| 백엔드, 서버, API, Spring, Java, 백 | Hao-Day-Backend | `../Hao-Day-Backend` |
| 인프라, 배포, Docker, Nginx, 서버설정 | Hao-Day-Infra | `../Hao-Day-Infra` |

### 예시
- "앱에서 로그인 화면 수정해줘" → `../Hao-Day-App/` 작업
- "백엔드 User API 추가해줘" → `../Hao-Day-Backend/` 작업
- "인프라 포트 변경해줘" → `../Hao-Day-Infra/` 작업
- "API 응답 필드 추가하고 앱에서도 반영해줘" → 크로스 레포 작업

---

## 아키텍처 개요

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Hao-Day-App   │────▶│  Hao-Day-Infra  │────▶│ Hao-Day-Backend │
│    (Flutter)    │     │  (Nginx/Docker) │     │  (Spring Boot)  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
   모바일 앱 UI           리버스 프록시              REST API
   상태 관리             로드밸런싱                비즈니스 로직
   API 클라이언트         SSL/TLS                  데이터베이스
```

### 서버 구성
| IP | 역할 |
|------|--------|
| 192.168.219.104 | Storage Server (RustFS) |
| 192.168.219.105 | Development Server |
| 192.168.219.106 | Production Server |
| 192.168.219.107 | Nginx Proxy / Local Registry |

### 도메인 구조
| 환경 | 도메인 | 대상 |
|------|--------|------|
| Production | api.haotoday.com | 106:18080 |
| Development | api-dev.haotoday.com | 105:18080 |

### 배포 흐름
```
GitHub Push → GitHub Actions → Local Registry (107) → Watchtower
    │
    ├── develop push → :dev 태그 → 105번 서버 (Dev)
    └── main push    → :latest 태그 → 106번 서버 (Prod)
```

---

## 연동 포인트

### API 변경 시 영향

| 변경 위치 | 영향받는 곳 | 확인 사항 |
|-----------|-------------|-----------|
| Backend API 엔드포인트 | App의 API 클라이언트 | URL 경로 수정 |
| Backend 응답 DTO | App의 model 클래스 | 필드 매핑 |
| Backend 포트 변경 | Infra의 nginx.conf | upstream 설정 |
| Infra SSL 인증서 | App의 .env | 도메인 확인 |

### 환경변수 동기화

| 레포 | 파일 |
|------|------|
| Backend | `application-{local,dev,prod}.yml` |
| Infra | `docker-compose.yml`, `.env` |
| App | `.env`, `.env.local`, `.env.dev`, `.env.prod` |

---

## 공통 Commands (이 워크스페이스에서 사용)

| Command | 설명 | 예시 |
|---------|------|------|
| `/plan-issue {번호}` | 이슈를 서브이슈로 분리 | `/plan-issue 12` |
| `/sync-docs` | 변경 후 문서 동기화 | `/sync-docs` |
| `/create-issue` | 대화 기반 이슈 생성 | `/create-issue` |
| `/commit-and-pr` | 커밋 및 PR 생성 | `/commit-and-pr #15` |
| `/cross-repo` | 여러 레포 동시 작업 | `/cross-repo API 필드 추가` |

**레포 자동 감지**: commands 실행 시 작업 중인 레포를 자동으로 감지합니다.
상위 폴더에서 실행 시 어떤 레포에서 작업할지 질문합니다.

---

## 레포별 특화 Commands

### Backend (`../Hao-Day-Backend/`)
| Command | 설명 |
|---------|------|
| `/check-architecture` | Java 멀티모듈 아키텍처 검증 |
| `/code-review` | Spring/JPA 코드 리뷰 |
| `/create-test-code {클래스명}` | JUnit 테스트 생성 |
| `/run-test` | Gradle 테스트 실행 |
| `/create-migration {설명}` | Flyway 마이그레이션 생성 |

### Infra (`../Hao-Day-Infra/`)
| Command | 설명 |
|---------|------|
| `/test-nginx` | Nginx 설정 테스트 |

### App (`../Hao-Day-App/`)
| Command | 설명 |
|---------|------|
| `/check-architecture` | Flutter 아키텍처 검증 |
| `/daily-review {날짜}` | 일일 학습 문서 생성 |
| `/figma-to-issues {URL}` | 피그마 → 이슈 변환 |
| `/parallel-work {작업목록}` | Git worktree 병렬 작업 |

---

## 크로스 레포 작업 가이드

### API 변경 예시

1. Backend에서 API 수정 (`../Hao-Day-Backend/`)
2. App에서 model 클래스 업데이트 (`../Hao-Day-App/`)
3. 각 레포에서 커밋 & PR

### 환경변수 변경 예시

1. 변경 사항 명세 작성
2. Backend → Infra → App 순서로 업데이트
3. 각 레포에서 커밋 & PR

### `/cross-repo` 사용 예시

```
/cross-repo User API에 profileImageUrl 필드 추가

→ 영향 분석:
  - Backend: UserResponse.java 필드 추가
  - App: user_model.dart 필드 추가
→ 작업 순서 제안
→ 순차 실행
```

---

## 사용 방법

### 1. 통합 모드 (이 폴더에서)
```bash
cd /Users/popo/Hao-Day/Hao-Day-Workspace
claude

# 모든 레포 접근 가능
# 자연어로 레포 전환: "앱에서 ~", "백엔드에서 ~"
```

### 2. 개별 모드 (각 레포에서)
```bash
cd /Users/popo/Hao-Day/Hao-Day-Backend
claude
# Backend 특화 commands만 사용
```

---

## 각 레포 상세 문서

- [Backend CLAUDE.md](../Hao-Day-Backend/CLAUDE.md) - 멀티모듈 아키텍처, 패턴
- [Infra CLAUDE.md](../Hao-Day-Infra/CLAUDE.md) - 서버 구성, 배포 흐름
- [App CLAUDE.md](../Hao-Day-App/CLAUDE.md) - Flutter 아키텍처, 화면 구성

---

## 주의사항

### Git 관리
- 각 레포는 **독립된 git repository**입니다
- 이 Workspace 레포는 설정 파일만 포함합니다
- 코드 변경은 각 레포에서 개별 커밋 & PR합니다

### 브랜치 전략
| 레포 | 기본 브랜치 | PR 대상 |
|------|-------------|---------|
| Backend | develop | develop → main |
| Infra | main | main |
| App | main | main |

### 민감 정보
- `.env` 파일은 각 레포의 `.gitignore`에 포함
- API 키, 비밀번호는 절대 커밋하지 않음
