# Create GitHub Issue

대화 내용을 분석하여 GitHub Issue를 생성합니다.

## 레포지토리 자동 감지

현재 작업 디렉토리에서 레포지토리를 자동으로 감지합니다:

```bash
# 현재 디렉토리 확인
pwd

# 레포 판단
# - Hao-Day-Backend 포함 → Another-Hao-Day/Hao-Day-Backend
# - Hao-Day-Infra 포함 → Another-Hao-Day/Hao-Day-Infra
# - Hao-Day-App 포함 → Another-Hao-Day/Hao-Day-App
# - Hao-Day-Workspace 또는 상위 폴더 → 사용자에게 질문
```

**상위 폴더에서 실행 시**: 대화 내용을 분석하여 관련 레포를 추론하거나, 불분명하면 사용자에게 질문하세요.

## 인자

- `$ARGUMENTS`: (선택) 이슈 제목 또는 추가 설명

## 실행 단계

### 1. 대화 컨텍스트 분석

이전 대화에서 다음을 파악하세요:
- 무엇을 해야 하는지 (기능 추가, 버그 수정, 리팩토링 등)
- 왜 필요한지 (배경, 문제점)
- 어디를 수정해야 하는지 (관련 파일, 컴포넌트)

### 2. 이슈 타입 결정

| 타입 | 라벨 | 제목 prefix |
|------|------|-------------|
| 새 기능 | `enhancement` | `feat:` |
| 버그 수정 | `bug` | `fix:` |
| 리팩토링 | `refactor` | `refactor:` |
| 문서 | `documentation` | `docs:` |
| 설정/인프라 | `chore` | `chore:` |

### 3. 레포별 이슈 템플릿

#### Backend
```bash
gh issue create -R Another-Hao-Day/Hao-Day-Backend \
  --title "{prefix}: {간결한 제목}" \
  --body "$(cat <<'EOF'
## 개요
{무엇을 해야 하는지 1-2문장}

## 배경
{왜 필요한지, 현재 문제점}

## 작업 내용
- [ ] {구체적인 작업 1}
- [ ] {구체적인 작업 2}
- [ ] {구체적인 작업 3}

## 관련 파일
- `{파일경로1}`
- `{파일경로2}`

## 관련 모듈
- [ ] support
- [ ] domain
- [ ] infrastructure
- [ ] api

## 참고사항
{추가 정보, 제약사항 등}
EOF
)" \
  --label "{라벨}"
```

#### Infra
```bash
gh issue create -R Another-Hao-Day/Hao-Day-Infra \
  --title "{prefix}: {간결한 제목}" \
  --body "$(cat <<'EOF'
## 개요
{무엇을 해야 하는지 1-2문장}

## 배경
{왜 필요한지}

## 작업 내용
- [ ] {구체적인 작업 1}
- [ ] {구체적인 작업 2}

## 영향 범위
- [ ] Nginx 설정
- [ ] Docker Compose
- [ ] GitHub Actions
- [ ] 서버 설정

## 참고사항
{추가 정보}
EOF
)"
```

#### App
```bash
gh issue create -R Another-Hao-Day/Hao-Day-App \
  --title "{prefix}: {간결한 제목}" \
  --body "$(cat <<'EOF'
## 개요
{무엇을 해야 하는지 1-2문장}

## 배경
{왜 필요한지}

## 작업 내용
- [ ] {구체적인 작업 1}
- [ ] {구체적인 작업 2}

## 관련 파일
- `{파일경로1}`

## UI 참고
{Figma 링크 또는 스크린샷}

## 참고사항
{추가 정보}
EOF
)"
```

### 4. 결과 안내

이슈 생성 후 사용자에게 알려주세요:
- 이슈 번호와 URL
- 작업 시작 방법 안내: "작업 완료 후 `/commit-and-pr #{이슈번호}` 로 PR을 생성하세요"

## 예시

### 입력
```
사용자: User API에 프로필 이미지 업로드 기능이 필요해
사용자: /create-issue
```

### 출력
```
Issue #15 생성 완료!
- 레포: Hao-Day-Backend
- 제목: feat: User API 프로필 이미지 업로드 기능 추가
- URL: https://github.com/.../issues/15

작업 완료 후 `/commit-and-pr #15` 로 PR을 생성하세요.
```

## 주의사항

- 제목은 50자 이내로 간결하게
- 작업 내용은 체크리스트로 구체적으로
- `$ARGUMENTS`가 있으면 해당 내용을 우선 반영
- 대화 컨텍스트가 불충분하면 사용자에게 질문
- 크로스 레포 작업이 필요하면 각 레포에 이슈 생성 제안
