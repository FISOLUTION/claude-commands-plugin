# claude-commands-plugin

에프아이솔루션 Claude Code 공용 도구 플러그인

## 설치 방법

### 방법 1: 마켓플레이스로 설치 (권장)

```bash
# 마켓플레이스 추가
/plugin marketplace add FISOLUTION/claude-commands-plugin

# 플러그인 설치
/plugin install common-tools@fisolution-marketplace
```

### 방법 2: 팀 프로젝트 자동 설치 설정

각 레포지토리의 `.claude/settings.json`에 다음을 추가하면 팀원들이 해당 레포지토리를 trust할 때 자동으로 마켓플레이스와 플러그인이 설치됩니다:

```json
{
  "extraKnownMarketplaces": {
    "fisolution-marketplace": {
      "source": {
        "source": "github",
        "repo": "FISOLUTION/claude-commands-plugin"
      }
    }
  },
  "enabledPlugins": {
    "common-tools@fisolution-marketplace": true
  }
}
```

## 제공 커맨드

### `/commit`

누적된 변경사항을 Conventional Commits 형식으로 적절히 나누어 커밋합니다.

**특징:**
- Conventional Commits 형식 사용 (`feat:`, `fix:`, `refactor:` 등)
- 한국어로 간결한 커밋 메시지 작성
- 한 줄 커밋 메시지 (본문 없음)
- 변경사항을 논리적 단위로 분할하여 커밋

**예시:**
```
feat: shadcn field 컴포넌트 추가
fix: false 값이 엑셀에서 올바르게 표현되지 않는 문제 수정
refactor: 이미지 정보 필드를 추가 정보 필드로 변경
```

### `/pr`

현재 브랜치의 변경사항으로 GitHub PR을 생성합니다.

**특징:**
- 브랜치명 검증 (`{issue-type}/{issue-number}-{description}` 형식)
- PR 템플릿 자동 적용
- `CONTEXT.md` 파일이 있으면 참고사항에 자동 포함
- `ISSUE.md` 파일 연동
- 브랜치명이 형식에 맞지 않으면 `create-issue` 에이전트를 호출하여 이슈 생성

## 제공 에이전트

### `create-issue`

GitHub 이슈를 확인하거나 생성하는 서브에이전트입니다.

**기능:**
- 이슈 번호가 있으면 조회, 없으면 새 이슈 생성
- 모호한 요구사항을 명확하고 실행 가능한 형태로 구체화
- 결과를 `ISSUE.md` 파일로 저장

**이슈 타입:**
- `feature` - 새로운 기능
- `fix` - 버그 수정
- `refactor` - 리팩토링
- `docs` - 문서
- `chore` - 기타 작업
- `test` - 테스트

### `create-context`

`CONTEXT.md` 파일을 생성하는 서브에이전트입니다.

**기능:**
- `ISSUE.md` 파일을 읽어 세션 간 공유 컨텍스트 파일 생성
- 작업 정보, 요구사항, 구현 계획, 세션 기록 섹션 포함
- 메인 에이전트가 구현 계획을 수립할 수 있도록 템플릿 제공

## 디렉토리 구조

```
claude-commands-plugin/
├── .claude-plugin/
│   ├── marketplace.json    # 마켓플레이스 정의
│   ├── plugin.json         # 플러그인 매니페스트
│   ├── agents/
│   │   ├── create-issue.md     # 이슈 생성/확인 에이전트
│   │   └── create-context.md   # CONTEXT.md 생성 에이전트
│   └── commands/
│       ├── commit.md           # /commit 커맨드
│       └── pr.md               # /pr 커맨드
└── README.md
```

## 라이선스

MIT
