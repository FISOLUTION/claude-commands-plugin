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

2. `pr_body.md` 파일을 Read 도구로 읽고, Edit 도구로 다음 섹션을 작성:
   - **작업 내용 및 특이사항**: 변경사항을 분석하여 주요 내용 요약 (3-5 bullet points)
   - **참고사항**: `CONTEXT.md`가 존재하면 해당 내용(요구사항, 구현 계획, 세션 기록)을 **요약**하여 포함. 리뷰어가 알아야 할 컨텍스트 정리 (설계 결정 이유, 논의된 대안 등)
   - **기타**: `-` 유지 (다음 단계에서 자동 첨부)

3. `CONTEXT.md`가 존재하면, 기타 섹션에 전체 내용을 `<details>` 태그로 첨부:
   ```bash
   if [ -f CONTEXT.md ]; then
     echo -e '\n<details>\n<summary>CONTEXT.md 전체 내용</summary>\n' >> pr_body.md
     echo '```markdown' >> pr_body.md
     cat CONTEXT.md >> pr_body.md
     echo -e '\n```\n</details>' >> pr_body.md
   fi
   ```

### 3단계: PR 생성
작성된 본문 파일로 PR을 생성합니다:
```bash
gh pr create --base main --body-file pr_body.md --title "$(git rev-parse --abbrev-ref HEAD)"
```
- `--body-file pr_body.md`: 2단계에서 작성된 본문 파일 사용
- `--title`: 현재 브랜치 이름을 PR 제목으로 사용 (이후 PR Automation이 덮어씀)
