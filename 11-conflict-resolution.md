# 11. 충돌 해결

[← 목차로 돌아가기](./README.md) | [← 이전: 태그](./10-tag.md)

---

## 충돌이란?

두 브랜치가 **같은 파일의 같은 부분**을 서로 다르게 수정했을 때, Git이 자동으로 합칠 수 없는 상태를 말한다.

### 충돌이 발생하는 상황

- `git merge` 시
- `git rebase` 시
- `git pull` 시 (내부적으로 merge 또는 rebase)
- `git cherry-pick` 시
- `git stash pop/apply` 시

---

## 충돌 표시 형식

충돌이 발생하면 파일에 아래와 같은 마커가 삽입된다:

```
<<<<<<< HEAD
현재 브랜치의 코드 (내 변경사항)
=======
병합 대상 브랜치의 코드 (상대방의 변경사항)
>>>>>>> feature/login
```

### 3-way merge 표시 (rebase 또는 diff3 설정 시)

```
<<<<<<< HEAD
현재 코드
||||||| merged common ancestors
공통 조상의 원본 코드
=======
상대방의 코드
>>>>>>> feature/login
```

diff3 스타일을 기본으로 설정하면 충돌 원인 파악이 쉬워진다:

```bash
git config --global merge.conflictstyle diff3
```

---

## 충돌 해결 절차

### 1단계: 충돌 상태 확인

```bash
# 충돌 파일 목록 확인
git status

# 출력 예시:
# Unmerged paths:
#   both modified:   src/auth/login.js
#   both modified:   src/utils/helper.js
```

### 2단계: 충돌 파일 수정

파일을 열고 충돌 마커(`<<<<<<<`, `=======`, `>>>>>>>`)를 제거하며 올바른 코드로 수정한다.

```javascript
// 수정 전 (충돌 상태)
<<<<<<< HEAD
const timeout = 3000;
=======
const timeout = 5000;
>>>>>>> feature/login

// 수정 후 (해결)
const timeout = 5000;
```

### 3단계: 해결 완료 표시

```bash
# 해결된 파일 스테이징
git add src/auth/login.js

# 모든 충돌 해결 후 커밋
git commit                  # merge 시 자동 메시지 생성
```

---

## 상황별 충돌 해결

### merge 충돌

```bash
git merge feature/login
# CONFLICT (content): Merge conflict in src/auth/login.js

# 충돌 해결 후
git add .
git commit

# 또는 병합 취소
git merge --abort
```

### rebase 충돌

rebase는 커밋 하나씩 적용하므로, 여러 번 충돌이 발생할 수 있다.

```bash
git rebase main
# CONFLICT...

# 각 충돌마다:
# 1. 충돌 해결
# 2. git add .
# 3. git rebase --continue

# 전체 취소
git rebase --abort

# 현재 커밋 건너뛰기
git rebase --skip
```

### pull 충돌

```bash
git pull origin main
# CONFLICT...

# merge 방식 pull: merge와 동일하게 해결
git add .
git commit

# rebase 방식 pull (git pull --rebase):
git add .
git rebase --continue
```

### cherry-pick 충돌

```bash
git cherry-pick abc1234
# CONFLICT...

git add .
git cherry-pick --continue

# 또는 취소
git cherry-pick --abort
```

### stash pop 충돌

```bash
git stash pop
# CONFLICT...

# stash pop 충돌 시 stash는 삭제되지 않는다
git add .
git commit

# 해결 후 stash 수동 삭제
git stash drop
```

---

## 충돌 해결 도구

### VS Code

VS Code에서 충돌 파일을 열면 인라인 버튼이 표시된다:

- **Accept Current Change**: 내 변경사항 유지
- **Accept Incoming Change**: 상대방 변경사항 유지
- **Accept Both Changes**: 양쪽 모두 유지
- **Compare Changes**: 나란히 비교

### CLI 도구

```bash
# 머지 도구 실행
git mergetool

# 머지 도구 설정
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'
```

---

## 충돌 해결 전략

### ours / theirs

자동으로 한쪽 변경을 선택할 수 있다.

```bash
# merge 시: 충돌 시 현재 브랜치 우선
git merge -X ours feature/login

# merge 시: 충돌 시 대상 브랜치 우선
git merge -X theirs feature/login

# 파일 단위로 한쪽 선택
git checkout --ours path/to/file.txt     # 현재 브랜치 버전
git checkout --theirs path/to/file.txt   # 대상 브랜치 버전
git add path/to/file.txt

# rebase 시 (주의: ours/theirs 의미가 반대)
git rebase -X ours main      # main의 변경 우선
git rebase -X theirs main    # 현재 브랜치 변경 우선
```

> **주의**: rebase에서 `ours`와 `theirs`의 의미가 merge와 반대이다. rebase는 대상 브랜치 위에 현재 커밋을 다시 적용하므로, "ours"가 rebase 대상(main)이 된다.

---

## 충돌 예방

```bash
# 자주 최신 코드 가져오기
git fetch origin
git rebase origin/main       # 또는 merge

# PR 올리기 전 충돌 확인
git fetch origin
git merge --no-commit --no-ff origin/main
git merge --abort            # 확인만 하고 취소

# 작은 단위로 자주 커밋 & PR
# 같은 파일을 동시에 수정하지 않도록 팀원과 소통
```

---

## 실무 팁

### 복잡한 충돌 시 전략

```bash
# 1. 먼저 현재 상태 백업
git stash push -m "충돌 해결 전 백업"

# 2. merge 취소 후 diff 확인
git merge --abort
git diff main..feature/login

# 3. 변경 범위 파악 후 다시 merge
git merge feature/login
```

### rerere (Reuse Recorded Resolution)

한 번 해결한 충돌 패턴을 기억해서 같은 충돌이 다시 발생하면 자동으로 적용한다.

```bash
# rerere 활성화
git config --global rerere.enabled true

# 기록된 해결 방법 확인
git rerere status
git rerere diff
```

---

[다음: 협업 워크플로우 →](./12-collaboration-workflow.md)
