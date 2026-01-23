# Hao-Day Workspace

Hao-Day 프로젝트의 통합 워크스페이스 설정입니다.

## 목적

- 3개 레포지토리(Backend, Infra, App)를 통합 관리
- 공통 Claude commands 중앙화
- 크로스 레포 작업 지원

## 사용 방법

```bash
# 1. 이 폴더에서 Claude Code 실행
cd Hao-Day-Workspace
claude

# 2. 자연어로 레포 전환
"앱에서 로그인 화면 수정해줘"
"백엔드 API 추가해줘"
"인프라 포트 변경해줘"

# 3. 통합 commands 사용
/plan-issue 12        # 이슈 분리
/create-issue         # 이슈 생성
/commit-and-pr #15    # 커밋 & PR
/cross-repo "설명"    # 크로스 레포 작업
```

## 폴더 구조

```
Hao-Day/
├── Hao-Day-Workspace/    # 이 레포 (통합 설정)
│   ├── CLAUDE.md
│   └── .claude/commands/
├── Hao-Day-Backend/      # Java/Spring 백엔드
├── Hao-Day-Infra/        # Docker/Nginx 인프라
└── Hao-Day-App/          # Flutter 앱
```

## 관련 레포

- [Hao-Day-Backend](https://github.com/Another-Hao-Day/Hao-Day-Backend)
- [Hao-Day-Infra](https://github.com/Another-Hao-Day/Hao-Day-Infra)
- [Hao-Day-App](https://github.com/Another-Hao-Day/Hao-Day-App)
