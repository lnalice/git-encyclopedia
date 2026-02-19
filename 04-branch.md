# 04. 브랜치 관리

[← 목차로 돌아가기](./README.md) | [← 이전: 스테이징 & 커밋](./03-staging-and-commit.md)

---

## 브랜치 조회

```bash
# 로컬 브랜치 목록
git branch

# 원격 브랜치 목록
git branch -r

# 모든 브랜치 (로컬 + 원격)
git branch -a

# 브랜치별 마지막 커밋 확인
git branch -v

# 더 상세한 정보 (원격 추적 브랜치 포함)
git branch -vv

# 현재 브랜치에 병합된 브랜치 목록
git branch --merged

# 아직 병합되지 않은 브랜치 목록
git branch --no-merged

# 특정 패턴의 브랜치 검색
git branch --list "feature/*"

# 현재 브랜치 이름만 출력
git branch --show-current
```

---

## 브랜치 생성

```bash
# 브랜치 생성 (현재 위치에서)
git branch feature/login

# 특정 커밋에서 브랜치 생성
git branch feature/login abc1234

# 브랜치 생성 + 즉시 이동
git checkout -b feature/login
git switch -c feature/login          # Git 2.23+

# 원격 브랜치 기반으로 로컬 브랜치 생성 + 추적
git checkout -b feature/login origin/feature/login
git switch -c feature/login origin/feature/login

# 원격 브랜치와 동일한 이름이면 줄여서
git checkout --track origin/feature/login
git switch --track origin/feature/login

# 이력 없는 고아 브랜치 생성 (gh-pages, 빈 문서 브랜치 등)
git switch --orphan gh-pages
git checkout --orphan gh-pages   # 레거시
```

> **orphan 브랜치**: 기존 커밋 이력이 전혀 없는 독립 브랜치를 만든다. GitHub Pages용 `gh-pages` 브랜치나, 프로젝트와 무관한 파일을 별도 관리할 때 사용한다.

---

## 브랜치 이동 (전환)

```bash
# 다른 브랜치로 이동
git checkout develop
git switch develop                   # Git 2.23+

# 이전 브랜치로 돌아가기
git checkout -
git switch -

# 특정 커밋으로 이동 (Detached HEAD)
git checkout abc1234
```

> **switch vs checkout**: `git switch`는 브랜치 전환 전용 명령어로, `checkout`의 다양한 기능 중 브랜치 전환만 분리한 것이다.

---

## 브랜치 이름 변경

```bash
# 현재 브랜치 이름 변경
git branch -m new-name

# 특정 브랜치 이름 변경
git branch -m old-name new-name

# 원격 브랜치 이름 변경 (삭제 후 재생성)
git push origin --delete old-name
git push origin -u new-name
```

---

## 브랜치 삭제

```bash
# 로컬 브랜치 삭제 (병합 완료된 브랜치)
git branch -d feature/login

# 강제 삭제 (병합 여부 무관)
git branch -D feature/login

# 원격 브랜치 삭제
git push origin --delete feature/login
git push origin :feature/login        # 축약형

# 원격에서 삭제된 브랜치의 로컬 참조 정리
git fetch --prune
git remote prune origin               # 동일 효과
```

---

## 브랜치 추적 설정

```bash
# 현재 브랜치의 업스트림 설정
git branch --set-upstream-to=origin/feature/login

# push하면서 업스트림 설정
git push -u origin feature/login

# 업스트림 해제
git branch --unset-upstream
```

---

## 브랜치 네이밍 컨벤션

팀에서 일관된 브랜치 이름을 사용하면 관리가 편해진다.

| 접두사 | 용도 | 예시 |
|--------|------|------|
| `feature/` | 새 기능 개발 | `feature/social-login` |
| `fix/` | 버그 수정 | `fix/cart-quantity-bug` |
| `hotfix/` | 긴급 수정 (프로덕션) | `hotfix/payment-error` |
| `release/` | 릴리스 준비 | `release/v2.1.0` |
| `refactor/` | 리팩토링 | `refactor/user-service` |
| `docs/` | 문서 작업 | `docs/api-guide` |
| `test/` | 테스트 관련 | `test/auth-unit-tests` |
| `chore/` | 설정, 빌드 등 | `chore/eslint-config` |

### 이슈 번호 포함

```bash
feature/123-social-login
fix/456-cart-bug
```

---

## 브랜치 전략

### Git Flow

```
main ─────────────────────────────────── (프로덕션)
  └─ develop ─────────────────────────── (개발 통합)
       ├─ feature/login ──┘              (기능 개발)
       ├─ feature/payment ──┘
       └─ release/v1.0 ───── main        (릴리스 준비)
                              └─ hotfix ─ (긴급 수정)
```

### GitHub Flow (간소화)

```
main ──────────────────────────────────── (항상 배포 가능)
  ├─ feature/login ── PR ── merge ──┘     (기능 브랜치)
  └─ fix/bug-123 ──── PR ── merge ──┘     (수정 브랜치)
```

### Trunk-Based Development

```
main ──────────────────────────────────── (단일 브랜치)
  ├─ short-lived-branch (1~2일) ── merge
  └─ short-lived-branch (1~2일) ── merge
```

---

## worktree — 여러 브랜치 동시 작업

하나의 저장소에서 별도의 작업 디렉토리를 만들어 여러 브랜치를 동시에 체크아웃한다. stash하거나 커밋하지 않고도 다른 브랜치에서 작업할 수 있다.

```bash
# 새 worktree 생성 (기존 브랜치)
git worktree add ../hotfix-dir hotfix/urgent

# 새 브랜치를 만들면서 worktree 생성
git worktree add -b feature/experiment ../experiment

# worktree 목록 확인
git worktree list

# worktree 제거
git worktree remove ../hotfix-dir

# 정리 (삭제된 worktree 참조 제거)
git worktree prune
```

### 활용 사례

```bash
# 코드 리뷰하면서 현재 작업 유지
git worktree add ../review-pr42 origin/feature/pr-42
# ../review-pr42 디렉토리에서 리뷰, 현재 디렉토리에서 계속 작업

# 리뷰 완료 후 정리
git worktree remove ../review-pr42

# 핫픽스와 기능 개발 동시 진행
git worktree add ../hotfix hotfix/critical-bug
# ../hotfix에서 핫픽스, 현재 디렉토리에서 feature 계속
```

---

## 실무 팁

### 작업 시작 전 항상 최신 코드 받기

```bash
git switch main
git pull origin main
git switch -c feature/new-task
```

### 병합 전 대상 브랜치 최신화

```bash
git switch feature/my-task
git fetch origin
git rebase origin/main
# 또는
git merge origin/main
```

### 오래된 브랜치 일괄 정리

```bash
# 이미 병합된 로컬 브랜치 삭제 (main, develop 제외)
git branch --merged main | grep -v "main\|develop" | xargs git branch -d

# 원격에서 삭제된 브랜치 참조 정리
git fetch --prune
```

---

[다음: 원격 저장소 →](./05-remote.md)
