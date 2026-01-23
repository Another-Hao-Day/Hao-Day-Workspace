# 크로스 레포 작업 (cross-repo)

여러 레포지토리에 걸친 변경 작업을 수행합니다.

## 인자

- `$ARGUMENTS`: 작업 설명 (필수)

예시: `/cross-repo User API에 profileImageUrl 필드 추가`

## 수행 작업

### 1. 작업 분석

작업 설명을 분석하여 영향받는 레포지토리를 파악합니다:

| 키워드 | 영향 레포 |
|--------|-----------|
| API, 엔드포인트, DTO, Entity, Service | Backend |
| 화면, UI, 위젯, Provider, 모델 | App |
| Nginx, Docker, 배포, 포트, 서버 | Infra |
| API 필드 추가/변경 | Backend + App |
| 포트 변경 | Backend + Infra |
| 환경변수 변경 | Backend + Infra + App |

### 2. 영향 분석 리포트

```
## 크로스 레포 영향 분석

### 변경 요청
{사용자 입력}

### 영향받는 레포지토리

1. **Hao-Day-Backend** (../Hao-Day-Backend/)
   - 파일: {예상 파일 목록}
   - 변경: {변경 내용}

2. **Hao-Day-App** (../Hao-Day-App/)
   - 파일: {예상 파일 목록}
   - 변경: {변경 내용}

### 작업 순서 권장
1. Backend 먼저 수정 (API 계약 변경)
2. App에서 모델/클라이언트 업데이트
3. 각각 테스트 후 PR

진행할까요?
```

### 3. 승인 시 작업 수행

사용자 승인 후 각 레포에서 순차적으로 작업을 수행합니다.

#### 작업 순서 원칙
1. **Backend 먼저**: API 계약이 변경되는 경우
2. **Infra 먼저**: 포트/도메인이 변경되는 경우
3. **App 마지막**: 클라이언트 측 변경

#### 각 레포 작업 완료 후
- 해당 레포 디렉토리로 이동
- 변경 사항 커밋
- PR 생성 (필요시)
- 결과 요약

### 4. 최종 리포트

```
## 크로스 레포 작업 완료

### Backend
- 변경 파일: {파일 목록}
- PR: #{PR번호} {URL}

### App
- 변경 파일: {파일 목록}
- PR: #{PR번호} {URL}

### 다음 단계
1. Backend PR 머지 후 dev 배포 확인
2. App PR 머지
3. 통합 테스트
```

## 예시

### API 응답 필드 추가
```
사용자: /cross-repo User API에 profileImageUrl 필드 추가

Claude:
## 크로스 레포 영향 분석

### 변경 요청
User API에 profileImageUrl 필드 추가

### 영향받는 레포지토리

1. **Hao-Day-Backend**
   - `domain/src/.../User.java`: 엔티티 필드 추가
   - `api/src/.../UserResponse.java`: DTO 필드 추가
   - `domain/src/.../db/migration/V3__add_profile_image_url.sql`: 마이그레이션

2. **Hao-Day-App**
   - `lib/features/auth/data/models/user_model.dart`: 필드 추가
   - `lib/features/profile/presentation/screens/profile_screen.dart`: UI 반영

### 작업 순서
1. Backend 먼저 (API 계약 변경)
2. App에서 모델 업데이트

진행할까요?
```

### 포트 변경
```
사용자: /cross-repo 백엔드 포트를 8080에서 8081로 변경

Claude:
## 크로스 레포 영향 분석

### 영향받는 레포지토리

1. **Hao-Day-Backend**
   - `api/src/main/resources/application.yml`: server.port 변경

2. **Hao-Day-Infra**
   - `vm-management/nginx.conf`: upstream 포트 변경
   - `backend-dev/docker-compose.yml`: 포트 매핑 변경
   - `backend-prod/docker-compose.yml`: 포트 매핑 변경

### 작업 순서
1. Infra 먼저 (nginx upstream 준비)
2. Backend 포트 변경
3. 배포 후 확인

진행할까요?
```

## 주의사항

- 각 레포 변경은 별도 브랜치에서 수행
- Backend 변경 시 하위 호환성 유지 권장 (optional 필드 추가 등)
- 동시 배포가 필요한 경우 배포 순서 주의
- 큰 변경은 각 레포에 이슈를 먼저 생성 권장 (`/create-issue`)
- 변경 후 각 레포의 테스트 실행 권장

## 크로스 레포 작업 시 커밋 메시지 예시

```
feat: add profileImageUrl field to User API

Part of cross-repo change:
- Backend: Add field to entity and DTO
- App: Update user model (separate PR)

Refs #15

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```
