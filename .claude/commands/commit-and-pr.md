# Commit and Create PR (with Issue Link)

변경사항을 커밋하고 PR을 생성합니다. Issue 번호가 주어지면 자동으로 연결하고, 없으면 관련 이슈를 자동으로 찾아 제안합니다.

## 레포지토리 자동 감지 및 빌드 도구 결정

현재 작업 디렉토리에서 레포지토리와 빌드 도구를 자동으로 감지합니다:

| 레포 | 빌드/포맷 명령 | PR Base 브랜치 |
|------|----------------|----------------|
| Hao-Day-Backend | `./gradlew spotlessApply && ./gradlew build -x test` | develop |
| Hao-Day-Infra | (없음) | main |
| Hao-Day-App | `dart format .` | main |

## 인자

- `$ARGUMENTS`: (선택) Issue 번호 (예: `#15` 또는 `15`)

## 실행 단계

### 1. 변경사항 확인

```bash
git status
git diff --staged
git diff
```

변경사항이 없으면 사용자에게 알리고 종료하세요.

### 2. Issue 번호 확인

#### 2-1. 인자로 Issue 번호가 주어진 경우

`$ARGUMENTS`에서 Issue 번호를 추출하고 정보를 조회하세요:

```bash
gh issue view {이슈번호} -R {repo} --json title,body,labels
```

#### 2-2. 인자가 없는 경우 - 자동 이슈 매칭

다음 순서로 관련 이슈를 찾으세요:

**Step 1: 브랜치 이름에서 이슈 번호 추출 시도**

```bash
git branch --show-current
# 예: feature/#17-user-api → #17 추출
# 패턴: #숫자, issue-숫자, /숫자- 등
```

**Step 2: 브랜치에서 못 찾으면 열린 이슈 목록 조회**

```bash
# 열린 이슈 목록 조회 (최근 20개)
gh issue list -R {repo} --state open --limit 20 --json number,title,body,labels
```

**Step 3: 변경사항과 이슈 매칭**

다음 정보를 비교하여 관련성 점수를 계산:
- 변경된 파일 경로 vs 이슈 본문의 "관련 파일" 섹션
- 변경 내용 키워드 vs 이슈 제목/본문 키워드
- 이슈 라벨 vs 변경 타입 (feat↔enhancement, fix↔bug 등)

**Step 4: 사용자에게 선택 요청**

매칭된 이슈가 있으면 AskUserQuestion 도구로 선택지를 제시.

### 3. 브랜치 확인 및 생성

현재 브랜치가 `main` 또는 `develop`이면 새 브랜치를 생성하세요:

```bash
# 현재 브랜치 확인
git branch --show-current

# main/develop이면 새 브랜치 생성
# 브랜치명: {type}/issue-{번호}-{간단한설명}
# 예: feat/issue-15-add-profile-upload
git checkout -b {브랜치명}
```

### 4. 커밋 메시지 작성

변경사항을 분석하여 커밋 메시지를 작성하세요:

```
{type}: {간결한 설명}

{상세 설명 (선택)}

{Issue 연결이 있으면}
Refs #{이슈번호}

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

**커밋 타입:**
- `feat`: 새 기능
- `fix`: 버그 수정
- `refactor`: 리팩토링
- `docs`: 문서
- `chore`: 설정/빌드
- `test`: 테스트

### 5. 빌드 검증 및 코드 포맷팅 (레포별)

#### Backend
```bash
./gradlew spotlessApply
./gradlew build -x test
```

#### App
```bash
dart format .
```

#### Infra
빌드 단계 없음

빌드 실패 시 → 오류 내용 출력 후 **중단**

### 6. 커밋 및 푸시

```bash
git add -A
git commit -m "{메시지}"
git push -u origin {브랜치명}
```

### 7. PR 생성 (레포별)

#### Backend
```bash
gh pr create -R Another-Hao-Day/Hao-Day-Backend \
  --base develop \
  --title "{커밋 제목과 동일}" \
  --body "$(cat <<'EOF'
## Summary
{변경사항 요약 - 1~3개 bullet point}

## Changes
{주요 변경 파일 및 내용}

## 변경된 모듈
- [ ] support
- [ ] domain
- [ ] infrastructure
- [ ] api

## Issue
{Issue 연결이 있으면}
Closes #{이슈번호}

## Test Plan
- [ ] 빌드 성공 (`./gradlew build`)
- [ ] {추가 테스트 항목}

---
Generated with [Claude Code](https://claude.ai/claude-code)
EOF
)"
```

#### Infra / App
```bash
gh pr create -R {repo} \
  --base main \
  --title "{커밋 제목과 동일}" \
  --body "$(cat <<'EOF'
## Summary
{변경사항 요약}

## Changes
{주요 변경 파일 및 내용}

## Issue
{Issue 연결이 있으면}
Closes #{이슈번호}

## Test Plan
- [ ] {테스트 항목}

---
Generated with [Claude Code](https://claude.ai/claude-code)
EOF
)"
```

**중요**: Issue를 닫으려면 `Closes #번호`, 참조만 하려면 `Refs #번호` 사용

### 8. 결과 안내

PR 생성 후 사용자에게 알려주세요:
- PR 번호와 URL
- 연결된 Issue 정보 (있는 경우)
- 다음 단계 (리뷰 요청, 머지 등)

## 주의사항

- 커밋 전 빌드/포맷 검증 자동 실행
- 민감한 정보 (.env, 키 등) 커밋 주의
- PR 제목은 커밋 컨벤션 따르기
- Issue 연결 시 `Closes`와 `Refs` 구분하여 사용
- **Backend PR 대상은 develop, 나머지는 main**
- `.idea/`, `*.iml` 등 IDE 파일은 커밋 제외

## 자동 이슈 매칭 팁

- **브랜치 네이밍 권장**: `feature/#17-description` 형식으로 브랜치를 만들면 자동 추출됨
- **이슈 작성 시 관련 파일 명시**: 이슈 본문에 `## 관련 파일` 섹션을 추가하면 매칭 정확도 향상
- **매칭 우선순위**: 브랜치 이름 > 파일 경로 일치 > 키워드 일치 > 라벨 일치
