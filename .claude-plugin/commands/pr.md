---
allowed-tools: Bash(git:*), Bash(gh:*)
description: 현재 브랜치의 변경사항으로 GitHub PR을 생성합니다
---

## 컨텍스트

### Git 상태
- 현재 브랜치: !`git branch --show-current`
- 최근 커밋: !`git log --oneline -5`
- 변경사항 요약: !`git diff --stat origin/main...HEAD`

### PR 템플릿
!`gh repo view --json pullRequestTemplates --jq '.pullRequestTemplates[0].body' 2>/dev/null || echo "템플릿 없음"`

### 관련 파일
- 이슈 정보: @ISSUE.md
- 작업 컨텍스트: @CONTEXT.md

## PR 템플릿 확인

- GitHub이 레포지토리 및 조직 `.github` 레포의 템플릿을 자동으로 통합 제공
- 템플릿이 비어있으면 PR 생성을 중단하고 사용자에게 안내

## PR 생성 단계

### 1단계: 준비

#### 브랜치명 검증

현재 브랜치명이 `{issue-type}/{issue-number}-{description}` 형식인지 확인합니다.
- 올바른 형식 예시: `feature/60-add-branch-validation`, `chore/8-create-pr`

**브랜치명 형식이 맞지 않는 경우:**
1. `ISSUE.md` 파일이 존재하는지 확인합니다.
   - 시스템 지침으로 인해 이슈가 생성된 후에도 포맷에 맞지 않는 브랜치명이 만들어질 수 있습니다.
2. `ISSUE.md`가 **없는 경우에만** `create-issue` 서브에이전트를 호출합니다.
   - 이미 작업이 완료된 상태에서도 기존 코드 변경사항에는 영향을 주지 않습니다.
   - 이슈 생성/조회만 수행되고, 결과는 `ISSUE.md`에 저장됩니다.
3. `ISSUE.md`의 이슈 정보를 바탕으로 브랜치명을 변경합니다:
   ```bash
   git branch -m {branch-type}/{issue-number}-{short-description}
   # 예: git branch -m feature/12-add-user-auth
   ```
   - `branch-type`: 브랜치 타입 (feature, fix, refactor, docs, chore, test)
     - **중요**: 이슈 레이블이 `feat`이더라도 브랜치명에는 반드시 `feature`를 사용합니다 (feat → feature)
   - `issue-number`: 생성/조회된 이슈 번호
   - `short-description`: 작업 내용을 간략히 설명하는 영문 (kebab-case)
4. 브랜치명이 변경되면, 변경된 브랜치명으로 아래 단계를 계속 진행합니다.

#### 이슈 번호 추출 및 준비

- 현재 브랜치에서 이슈 번호 추출 (예: `chore/8-create-pr` → 이슈 번호 8)
- `git diff origin/main...HEAD`로 변경사항 검토
- 변경사항을 origin에 push: `git push -u origin HEAD`

### 2단계: PR 본문 작성
1. PR 템플릿을 `pr_body.md`로 저장:
   ```bash
   gh repo view --json pullRequestTemplates --jq '.pullRequestTemplates[0].body' > pr_body.md
   ```

2. `pr_body.md` 파일을 Read 도구로 읽고, Edit 도구로 아래 항목별 규칙에 따라 작성합니다.

#### 항목별 작성 규칙

**IMPORTANT**: 아래 규칙을 반드시 지킵니다. 각 항목은 "적을 내용이 없으면 비워둔다"가 기본값입니다.

**작업 내용 및 특이사항** (최상단 항목)
- 변경 내용과 변경 사항 이해에 꼭 필요한 특이 사항만 **5줄 이내** bullet point로 요약합니다.
- 각 줄은 한 문장으로 간결하게 작성합니다.
- 커밋 로그를 그대로 나열하지 않고, 변경 사항을 의미 단위로 묶어 요약합니다.

**참고사항**
- **코드만 보고는 이해하기 어려운 배경이 있을 때만** 작성합니다. (예: 특정 방식을 선택해야 했던 외부 제약, 코드에 드러나지 않는 비즈니스 요구사항)
- 작성하는 경우에도 **2줄 이내**로 요약합니다.
- 코드만 봐도 이해되는 변경이면 **아무것도 적지 않습니다.** 대화 과정에서 논의했다는 이유만으로 적지 않습니다.

**기타**
- **다음 이슈로 진행하기로 지정한 후속 작업**만 적습니다.
- 다음 이슈로 지정한 내용이 없으면 **아무것도 적지 않습니다.**
- `CONTEXT.md` 전문, 세션 기록, 테스트 방법 등 다른 내용을 채우지 않습니다.

#### 공통 금지 사항

- **코드 블록, 예제 코드, 함수 시그니처 등 상세 구현 내용을 명시하지 않습니다.** 변경된 파일/모듈과 동작 변화만 서술합니다.
- **이모지를 사용하지 않습니다.** (템플릿에 이미 포함된 이모지는 유지)
- **볼드(`**`), 이탤릭 등 강조 표기를 사용하지 않습니다.**
- **대화 과정에서 나눈 세밀한 작업 사유, 시행착오, 논의된 대안, 설계 토론 내용을 적지 않습니다.**
- 항목을 채우기 위해 내용을 만들어내지 않습니다. 비어 있는 항목은 템플릿의 `-`를 그대로 유지합니다.

3. 작성이 끝나면 `pr_body.md`를 다시 읽어 위 규칙(5줄 이내, 참고사항 2줄 이내, 빈 항목 유지, 강조/이모지/코드 없음)을 위반한 부분이 없는지 점검하고 수정합니다.

### 3단계: PR 생성
작성된 본문 파일로 PR을 생성합니다:
```bash
gh pr create --base develop --body-file pr_body.md --title "$(git rev-parse --abbrev-ref HEAD)"
```
- `--body-file pr_body.md`: 2단계에서 작성된 본문 파일 사용
- `--title`: 현재 브랜치 이름을 PR 제목으로 사용 (이후 PR Automation이 덮어씀)
