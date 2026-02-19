# 13. GitHub CLI (`gh`)

[← 목차로 돌아가기](./README.md) | [← 이전: 협업 워크플로우](./12-collaboration-workflow.md)

---

## GitHub CLI란?

터미널에서 GitHub의 주요 기능(PR, Issue, Release 등)을 직접 사용할 수 있는 공식 CLI 도구이다.

### 설치

```bash
# macOS
brew install gh

# Windows
winget install GitHub.cli

# Ubuntu/Debian
sudo apt install gh
```

### 인증

```bash
# 로그인 (브라우저 인증)
gh auth login

# 토큰으로 로그인
gh auth login --with-token < token.txt

# 로그인 상태 확인
gh auth status

# 로그아웃
gh auth logout
```

---

## Pull Request 관리

### PR 생성

```bash
# 대화형 PR 생성
gh pr create

# 제목과 본문 지정
gh pr create --title "feat: 로그인 기능" --body "소셜 로그인을 추가합니다"

# 대상 브랜치 지정
gh pr create --base develop

# 리뷰어, 라벨, 마일스톤 지정
gh pr create --reviewer "teammate1,teammate2" \
             --label "feature" \
             --milestone "v2.0"

# Draft PR
gh pr create --draft

# 웹 브라우저에서 생성
gh pr create --web
```

### PR 조회

```bash
# 내가 만든 PR 목록
gh pr list

# 리뷰 요청 받은 PR
gh pr list --search "review-requested:@me"

# 특정 상태
gh pr list --state open
gh pr list --state closed
gh pr list --state merged

# 라벨로 필터
gh pr list --label "bug"

# PR 상세 정보
gh pr view 42
gh pr view 42 --web         # 브라우저에서 열기

# 현재 브랜치의 PR
gh pr view

# PR의 diff 확인
gh pr diff 42

# PR 코멘트 보기
gh pr view 42 --comments
```

### PR 체크아웃

```bash
# PR 코드를 로컬에 체크아웃
gh pr checkout 42
```

### PR 리뷰

```bash
# 리뷰 승인
gh pr review 42 --approve

# 수정 요청
gh pr review 42 --request-changes --body "API 에러 핸들링이 필요합니다"

# 코멘트만
gh pr review 42 --comment --body "전반적으로 좋습니다"
```

### PR 병합

```bash
# 기본 merge
gh pr merge 42

# Squash merge
gh pr merge 42 --squash

# Rebase merge
gh pr merge 42 --rebase

# 병합 후 브랜치 삭제
gh pr merge 42 --squash --delete-branch

# 자동 병합 (CI 통과 후 자동 머지)
gh pr merge 42 --auto --squash
```

### PR 상태 변경

```bash
# PR 닫기
gh pr close 42

# Draft 해제 (Ready for review)
gh pr ready 42

# PR 재오픈
gh pr reopen 42
```

---

## Issue 관리

### Issue 생성

```bash
# 대화형 생성
gh issue create

# 제목과 본문 지정
gh issue create --title "버그: 로그인 실패" --body "Safari에서 로그인 시 500 에러 발생"

# 라벨, 담당자 지정
gh issue create --title "기능 요청" \
                --label "enhancement" \
                --assignee "@me"
```

### Issue 조회

```bash
# 이슈 목록
gh issue list

# 나에게 할당된 이슈
gh issue list --assignee "@me"

# 라벨로 필터
gh issue list --label "bug"

# 이슈 상세
gh issue view 15
gh issue view 15 --web

# 이슈 코멘트 보기
gh issue view 15 --comments
```

### Issue 상태 변경

```bash
# 이슈 닫기
gh issue close 15

# 이슈 재오픈
gh issue reopen 15

# 코멘트 추가
gh issue comment 15 --body "수정 완료. PR #42에서 확인 가능합니다."

# 라벨 추가/제거
gh issue edit 15 --add-label "in-progress"
gh issue edit 15 --remove-label "backlog"

# 담당자 변경
gh issue edit 15 --add-assignee "teammate"
```

---

## 저장소 관리

```bash
# 저장소 생성
gh repo create my-project --public
gh repo create my-project --private --clone

# 저장소 복제
gh repo clone user/repo

# 저장소 Fork
gh repo fork user/repo
gh repo fork user/repo --clone     # fork + clone

# 저장소 브라우저에서 열기
gh repo view --web

# 저장소 정보
gh repo view user/repo
```

---

## Release 관리

```bash
# 릴리스 생성
gh release create v1.0.0

# 제목, 노트 포함
gh release create v1.0.0 --title "v1.0.0" --notes "첫 정식 릴리스"

# 커밋 로그로 자동 노트 생성
gh release create v1.0.0 --generate-notes

# 파일 첨부
gh release create v1.0.0 ./dist/app.zip ./dist/app.tar.gz

# Draft 릴리스
gh release create v1.0.0 --draft

# 프리릴리스
gh release create v2.0.0-beta.1 --prerelease

# 릴리스 목록
gh release list

# 릴리스 보기
gh release view v1.0.0

# 릴리스 삭제
gh release delete v1.0.0
```

---

## Workflow (GitHub Actions)

```bash
# 워크플로우 목록
gh workflow list

# 워크플로우 실행
gh workflow run deploy.yml

# 파라미터와 함께 실행
gh workflow run deploy.yml -f environment=staging

# 실행 상태 확인
gh run list

# 특정 실행 상세
gh run view <run-id>

# 실행 로그 보기
gh run view <run-id> --log

# 실패한 실행 재시도
gh run rerun <run-id>

# 실행 중인 워크플로우 취소
gh run cancel <run-id>

# 실행 결과 watch (실시간)
gh run watch <run-id>
```

---

## Gist 관리

```bash
# Gist 생성
gh gist create file.txt

# 설명 추가
gh gist create file.txt --desc "유용한 스니펫"

# 비공개 Gist
gh gist create file.txt --public=false

# 여러 파일
gh gist create file1.txt file2.txt

# Gist 목록
gh gist list

# Gist 보기
gh gist view <gist-id>

# Gist 수정
gh gist edit <gist-id>
```

---

## GitHub API 호출

`gh api`를 사용하면 GitHub REST/GraphQL API를 직접 호출할 수 있다.

```bash
# REST API
gh api repos/owner/repo/pulls

# POST 요청
gh api repos/owner/repo/issues -f title="New Issue" -f body="Description"

# GraphQL
gh api graphql -f query='{ viewer { login } }'

# 페이지네이션 자동 처리
gh api repos/owner/repo/issues --paginate

# JSON 필터링 (jq 문법)
gh api repos/owner/repo/pulls --jq '.[].title'
```

---

## 실무 팁

### PR 생성부터 머지까지 한 번에

```bash
# 작업 완료 후
git push -u origin feature/my-task
gh pr create --title "feat: 새 기능" --body "설명" --reviewer "lead"

# CI 통과 후 자동 머지 설정
gh pr merge --auto --squash --delete-branch
```

### 이슈 기반 작업 시작

```bash
# 이슈 확인
gh issue view 42

# 이슈 기반 브랜치 생성 + 작업 + PR
git switch -c fix/42-login-bug
# ... 작업 ...
git push -u origin fix/42-login-bug
gh pr create --title "fix: 로그인 버그 수정" --body "Closes #42"
```

---

[다음: .gitignore & 파일 관리 →](./14-gitignore.md)
