# GitHub 이슈 구조화 (plan-issue)

큰 이슈를 논리적 작업 단위로 분리하여 서브이슈를 생성합니다.

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

**상위 폴더에서 실행 시**: 사용자에게 어떤 레포에서 작업할지 질문하세요.
- "backend" 또는 "백엔드" → Another-Hao-Day/Hao-Day-Backend
- "infra" 또는 "인프라" → Another-Hao-Day/Hao-Day-Infra
- "app" 또는 "앱" → Another-Hao-Day/Hao-Day-App

## 입력

이슈 번호 (필수):
$ARGUMENTS

예시: `/plan-issue 12` 또는 `/plan-issue #12`

## 수행 작업

1. **이슈 분석**
   - 이슈 내용 조회 (`gh issue view`)
   - 목표, 범위, 작업 항목 파악

2. **작업 단위 분리**
   - 논리적으로 독립된 작업 단위 식별
   - 의존성 순서 고려
   - 각 서브이슈의 범위 정의

3. **서브이슈 생성**
   - 각 작업 단위별 이슈 생성 (`gh issue create`)
   - 표준 형식 적용 (목표, 체크리스트, Parent Issue)

4. **Parent-Child 연결**
   - GraphQL API로 서브이슈 연결
   - 메인 이슈 본문에 서브이슈 목록 추가

## 실행 절차

1. 레포지토리 감지 (자동 또는 질문)
2. 이슈 번호 파싱 (# 제거)
3. `gh issue view {번호} -R {repo}`로 이슈 내용 조회
4. 이슈 내용 분석하여 작업 단위 분리
5. 사용자에게 분리 계획 제시 및 확인
6. 승인 시 서브이슈 생성:
   ```bash
   gh issue create -R {repo} --title "feat: {작업명}" --body "..."
   ```
7. GraphQL API로 서브이슈 연결:
   ```bash
   # node_id 조회
   gh api repos/{owner}/{repo}/issues/{number} --jq '.node_id'

   # sub-issue 연결
   gh api graphql -f query='
   mutation {
     addSubIssue(input: {issueId: "{parent_node_id}", subIssueId: "{child_node_id}"}) {
       issue { title }
     }
   }'
   ```
8. 메인 이슈 본문 업데이트 (서브이슈 목록 추가)
9. 결과 요약 출력

## 서브이슈 템플릿

```markdown
## 목표
{작업 목표 1-2문장}

## 체크리스트
- [ ] {구체적 작업 항목 1}
- [ ] {구체적 작업 항목 2}
- [ ] {구체적 작업 항목 3}

## 참고사항
{필요시 추가 정보}

## Parent Issue
#{메인이슈번호}
```

## 분리 기준

- **독립 실행 가능**: 다른 작업 없이 단독 완료 가능
- **명확한 완료 조건**: 완료 여부 판단 가능
- **적절한 크기**: 1-4시간 내 완료 가능한 단위
- **단일 책임**: 하나의 목적에 집중

## 주의사항

- 서브이슈 생성 전 사용자 확인 필수
- 기존 서브이슈가 있으면 중복 생성 방지
- 레포별 라벨 체계 확인 후 적용
