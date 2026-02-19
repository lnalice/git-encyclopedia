# 12. 협업 워크플로우

[← 목차로 돌아가기](./README.md) | [← 이전: 충돌 해결](./11-conflict-resolution.md)

---

## Pull Request (PR) 기반 협업

대부분의 팀에서 사용하는 표준 협업 방식이다.

### 기본 흐름

```bash
# 1. 최신 main 브랜치 받기
git switch main
git pull origin main

# 2. 작업 브랜치 생성
git switch -c feature/user-profile

# 3. 작업 & 커밋
git add .
git commit -m "feat(profile): 프로필 편집 기능 구현"

# 4. 원격에 push
git push -u origin feature/user-profile

# 5. GitHub에서 PR 생성 (또는 gh CLI 사용)
# 6. 코드 리뷰 진행
# 7. 리뷰 반영 후 추가 커밋
git add .
git commit -m "refactor: 리뷰 반영 - 유효성 검사 분리"
git push

# 8. PR 승인 & 병합
# 9. 로컬 정리
git switch main
git pull origin main
git branch -d feature/user-profile
```

---

## Fork 기반 협업 (오픈소스)

원본 저장소에 직접 push 권한이 없을 때 사용한다.

```bash
# 1. GitHub에서 Fork
# 2. Fork한 저장소 clone
git clone git@github.com:my-account/project.git
cd project

# 3. 원본 저장소를 upstream으로 등록
git remote add upstream https://github.com/original/project.git

# 4. 작업 브랜치 생성
git switch -c fix/typo-in-readme

# 5. 작업 & 커밋 & push (내 fork에)
git add .
git commit -m "docs: README 오타 수정"
git push -u origin fix/typo-in-readme

# 6. GitHub에서 원본 저장소로 PR 생성

# 7. 원본 저장소 동기화 (주기적으로)
git fetch upstream
git switch main
git merge upstream/main
git push origin main
```

---

## 코드 리뷰 관련

### 리뷰어로서 PR 코드 로컬에서 확인

```bash
# PR 브랜치 가져오기
git fetch origin pull/42/head:pr-42
git switch pr-42

# 또는 팀원의 브랜치 직접 체크아웃
git fetch origin
git switch -c review/feature origin/feature/user-profile

# 확인 후 정리
git switch main
git branch -D pr-42
```

### 리뷰 반영 후 커밋 정리

```bash
# 리뷰 반영 커밋들을 squash
git rebase -i HEAD~3
# pick  abc1234 feat: 프로필 기능
# fixup def5678 fix: 리뷰 반영 1
# fixup ghi9012 fix: 리뷰 반영 2

git push --force-with-lease
```

---

## 병합 전략 (GitHub PR 옵션)

### Merge Commit (기본)

```bash
# 모든 커밋 이력과 머지 커밋이 남는다
git merge --no-ff feature/branch
```

- 브랜치 이력이 완전히 보존됨
- 머지 커밋으로 "언제 병합했는지" 명확

### Squash and Merge

```bash
# 모든 커밋이 하나로 합쳐진다
git merge --squash feature/branch
git commit -m "feat: 프로필 기능 (#42)"
```

- main 이력이 깔끔해짐
- PR 단위로 하나의 커밋
- 세부 커밋 이력은 PR에서만 확인 가능

### Rebase and Merge

```bash
# 커밋들이 main 위에 순서대로 재배치된다
git rebase main feature/branch
git switch main
git merge --ff-only feature/branch
```

- 선형 이력 유지
- 각 커밋이 독립적으로 보존됨
- 커밋 해시가 변경됨

### 어떤 전략을 선택할까?

| 전략 | 적합한 경우 |
|------|-----------|
| Merge Commit | 이력 보존이 중요한 프로젝트 |
| Squash and Merge | 커밋 단위가 작고 PR 단위 관리가 편한 경우 |
| Rebase and Merge | 깔끔한 선형 이력을 원하는 경우 |

---

## 협업 시 자주 쓰는 패턴

### 작업 시작 전 루틴

```bash
git switch main
git pull origin main
git switch -c feature/new-task
```

### 장기 작업 브랜치 최신화

```bash
git fetch origin
git rebase origin/main
# 충돌 해결 후
git push --force-with-lease
```

### PR 머지 후 정리

```bash
git switch main
git pull origin main
git branch -d feature/merged-branch     # 로컬 삭제
git fetch --prune                         # 원격 참조 정리
```

### 팀원 브랜치에서 이어서 작업

```bash
git fetch origin
git switch -c feature/teammate-task origin/feature/teammate-task
# 작업 & 커밋
git push origin feature/teammate-task
```

---

## CODEOWNERS 설정

특정 파일/디렉토리에 대한 리뷰어를 자동 지정한다. `.github/CODEOWNERS` 파일을 생성한다.

```
# 전체 기본 리뷰어
* @team-lead

# 디렉토리별 담당자
/src/auth/       @auth-team
/src/payment/    @payment-team
/docs/           @tech-writer

# 특정 파일
package.json     @devops-team
Dockerfile       @devops-team
```

---

## PR 템플릿

`.github/PULL_REQUEST_TEMPLATE.md` 파일을 만들면 PR 생성 시 자동으로 적용된다.

```markdown
## 변경 사항
<!-- 이 PR에서 변경한 내용을 설명하세요 -->

## 변경 이유
<!-- 왜 이 변경이 필요한지 설명하세요 -->

## 테스트
- [ ] 유닛 테스트 추가/수정
- [ ] 로컬에서 테스트 완료
- [ ] 스테이징 환경에서 확인

## 스크린샷 (선택)
<!-- UI 변경 시 스크린샷 첨부 -->

## 관련 이슈
Closes #이슈번호
```

---

## 패치 기반 협업 (format-patch / am)

push 권한이 없거나, 이메일/파일로 변경사항을 전달해야 할 때 사용한다. 리눅스 커널 등 대규모 오픈소스에서 표준 방식으로 쓰인다.

### 패치 생성

```bash
# 최근 커밋 1개를 패치 파일로 생성
git format-patch -1

# 최근 n개 커밋
git format-patch -3
# 0001-feat-로그인.patch
# 0002-fix-유효성검사.patch
# 0003-test-테스트추가.patch

# 특정 범위
git format-patch main..feature/login

# 출력 디렉토리 지정
git format-patch -3 -o ./patches/

# 하나의 파일로 합치기
git format-patch main..feature/login --stdout > all-changes.patch
```

### 패치 적용

```bash
# git am — 커밋 이력(작성자, 메시지)까지 보존하여 적용
git am 0001-feat-로그인.patch
git am ./patches/*.patch           # 여러 패치 순서대로 적용

# 충돌 발생 시
git am --abort                     # 취소
git am --skip                      # 건너뛰기
git am --continue                  # 충돌 해결 후 계속

# git apply — 커밋 없이 변경사항만 적용
git apply changes.patch

# 적용 가능 여부만 확인 (dry run)
git apply --check changes.patch

# 3-way merge로 적용 (충돌 해결 용이)
git am -3 0001-feat-로그인.patch
```

### 패치 활용 시나리오

```bash
# 1. push 권한 없는 팀원이 패치 전달
git format-patch -1 -o ~/Desktop/
# → 이메일이나 메신저로 .patch 파일 전달

# 2. 받는 사람이 적용
git am ~/Downloads/0001-feat-로그인.patch

# 3. 보안 환경에서 코드 전달 (네트워크 차단된 환경)
git format-patch main..HEAD --stdout > updates.patch
# USB로 전달 후
git am updates.patch
```

---

## .mailmap — 작성자 정보 매핑

팀원이 이메일을 변경했거나, 같은 사람이 다른 이름/이메일로 커밋한 경우 이력을 통합한다. 프로젝트 루트에 `.mailmap` 파일을 생성한다.

```
# 형식: 올바른 이름 <올바른 이메일> 이전 이름 <이전 이메일>

# 이메일 변경
홍길동 <gildong@new.com> <gildong@old.com>

# 이름 통일
홍길동 <gildong@company.com> GilDong Hong <gildong@company.com>

# 이름 + 이메일 모두 변경
홍길동 <gildong@company.com> gildong <gildong123@gmail.com>
```

적용 후 `git log`, `git shortlog`, `git blame` 등에서 통합된 이름으로 표시된다.

```bash
# 확인
git shortlog -s -n
# 변경 전: "GilDong Hong"과 "홍길동"이 별도 표시
# 변경 후: "홍길동"으로 통합
```

> **실무 팁**: `.mailmap`을 저장소에 커밋해두면 팀 전체에 적용된다. GitHub의 Contributors 그래프에도 반영된다.

---

## Git Hooks — 팀 자동화

커밋, push 등의 이벤트에 스크립트를 자동 실행하여 코드 품질을 보장한다.

### 주요 훅

| 훅 | 실행 시점 | 활용 예 |
|----|-----------|---------|
| `pre-commit` | 커밋 전 | 린트, 포맷 검사, 테스트 |
| `commit-msg` | 커밋 메시지 작성 후 | 메시지 형식 검증 (Conventional Commits) |
| `pre-push` | push 전 | 전체 테스트 실행 |
| `post-merge` | merge/pull 후 | 의존성 자동 설치 |
| `pre-rebase` | rebase 전 | 보호 브랜치 체크 |

### pre-commit 훅 예시

```bash
#!/bin/sh
# .git/hooks/pre-commit

npm run lint
if [ $? -ne 0 ]; then
    echo "린트 에러 발생. 커밋이 중단됩니다."
    exit 1
fi
```

### 팀 전체 공유 (husky)

`.git/hooks/`는 저장소에 포함되지 않으므로, 팀원 모두에게 동일한 훅을 적용하려면 husky를 사용한다.

```bash
# husky 설치 & 초기화
npx husky init

# pre-commit 훅 설정
echo "npx lint-staged" > .husky/pre-commit

# commit-msg 훅으로 메시지 형식 검증
echo 'npx commitlint --edit "$1"' > .husky/commit-msg
```

### lint-staged — 변경된 파일만 린트

```bash
# package.json에 추가
# "lint-staged": {
#   "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
#   "*.css": ["stylelint --fix"]
# }
```

> **실무 권장**: `husky` + `lint-staged` + `commitlint` 조합으로 커밋 품질을 자동 관리하는 것이 현대적인 협업 표준이다.

---

## 브랜치 보호 규칙

GitHub Settings > Branches에서 설정한다.

| 규칙 | 설명 |
|------|------|
| Require PR before merge | 직접 push 금지, PR 필수 |
| Require approvals | 최소 리뷰 승인 수 설정 |
| Require status checks | CI 통과 필수 |
| Require linear history | rebase merge만 허용 |
| Include administrators | 관리자에게도 규칙 적용 |

---

[다음: GitHub CLI →](./13-github-cli.md)
