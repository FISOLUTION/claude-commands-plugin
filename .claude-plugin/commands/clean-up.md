---
allowed-tools: Bash(git:*), Bash(gh:*), Bash(rm:*), Bash(ls:*)
description: PR 병합 후 develop으로 이동하고 작업 브랜치를 정리합니다
---

## 컨텍스트

### Git 상태
- 현재 브랜치: !`git branch --show-current`
- 로컬 브랜치 목록: !`git branch`
- 작업 트리 상태: !`git status --short`
- 현재 브랜치 PR 상태: !`gh pr view --json number,state,mergedAt --jq '"#\(.number) \(.state) merged=\(.mergedAt)"' 2>/dev/null || echo "PR 없음"`

## 개요

작업이 종료되고 PR이 병합된 후 사용합니다. 작업 브랜치를 정리하고 `develop` 브랜치를 최신 상태로 맞춥니다.

## 실행 단계

**IMPORTANT**: 반드시 다음 순서를 따라야 합니다.

### 1단계: 사전 확인

1. 현재 브랜치가 `develop` 또는 `main`이면 삭제할 작업 브랜치가 없으므로 3단계(develop 최신화)만 수행합니다.
2. 작업 트리에 커밋되지 않은 변경사항이 있으면 **작업을 중단**하고 사용자에게 안내합니다. (stash나 삭제를 임의로 수행하지 않음)
3. 현재 브랜치의 PR 상태를 확인합니다.
   - `MERGED`가 아닌 경우(OPEN, CLOSED, PR 없음) **작업을 중단**하고 사용자에게 상태를 안내합니다.
   - 사용자가 명시적으로 정리를 요청한 경우에만 계속 진행합니다.

### 2단계: develop으로 이동 및 작업 브랜치 삭제

1. 삭제할 작업 브랜치명을 변수로 기억합니다.
   ```bash
   WORK_BRANCH=$(git branch --show-current)
   ```
2. `develop` 브랜치로 이동합니다.
   ```bash
   git checkout develop
   ```
3. 작업 브랜치를 로컬에서 삭제합니다.
   ```bash
   git branch -d "$WORK_BRANCH"
   ```
   - `-d`가 실패하면(병합되지 않은 커밋이 있다고 판단되는 경우) squash merge 등으로 인한 것일 수 있습니다. PR이 `MERGED` 상태임을 1단계에서 확인했으므로 `git branch -D "$WORK_BRANCH"`로 강제 삭제합니다.
4. 원격 브랜치가 남아 있으면 삭제합니다. (GitHub 설정으로 이미 삭제되어 있으면 무시)
   ```bash
   git push origin --delete "$WORK_BRANCH" 2>/dev/null || true
   ```

### 3단계: develop 최신화

1. 원격의 병합 사항을 반영합니다.
   ```bash
   git pull origin develop
   ```
2. 원격에서 삭제된 브랜치 참조를 정리합니다.
   ```bash
   git fetch --prune
   ```

### 4단계: 작업 파일 정리

작업 중 생성된 임시 파일이 남아 있으면 삭제합니다. (`.gitignore`에 포함되어 있어 커밋되지 않은 파일)
```bash
rm -f ISSUE.md CONTEXT.md pr_body.md
```

### 5단계: 최종 확인

- `git branch --show-current`가 `develop`인지 확인
- `git status`가 clean인지 확인
- `git log --oneline -3`으로 병합 커밋이 반영되었는지 확인
- 삭제한 브랜치명과 최신화 결과를 한 줄로 사용자에게 보고
