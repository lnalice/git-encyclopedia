# 06. 병합 & 리베이스

[← 목차로 돌아가기](./README.md) | [← 이전: 원격 저장소](./05-remote.md)

---

## merge — 브랜치 병합

```bash
# feature 브랜치를 main에 병합
git switch main
git merge feature/login

# 머지 커밋 메시지 직접 지정
git merge feature/login -m "Merge feature/login into main"

# fast-forward 병합만 허용 (분기 없으면 직진)
git merge --ff-only feature/login

# 항상 머지 커밋 생성 (이력 보존)
git merge --no-ff feature/login

# 병합 중단
git merge --abort

# 병합 결과 미리 보기 (실제 병합 안 함)
git merge --no-commit feature/login
git diff --cached                  # 결과 확인
git merge --abort                  # 취소
# 또는
git commit                         # 확인 후 커밋
```

### Fast-Forward vs No Fast-Forward

```
# Fast-Forward (--ff) — 기본값
# 분기가 없으면 포인터만 이동. 머지 커밋 없음.
main: A ─ B ─ C
                └─ feature: D ─ E
결과:  A ─ B ─ C ─ D ─ E (main)

# No Fast-Forward (--no-ff)
# 항상 머지 커밋을 생성하여 병합 이력을 남김.
main: A ─ B ─ C ─────── M (merge commit)
                └─ D ─ E ┘
```

> **실무 권장**: PR 기반 협업에서는 `--no-ff`로 병합 이력을 남기는 것이 좋다. GitHub의 "Merge pull request"는 기본적으로 `--no-ff`이다.

---

## rebase — 커밋 재배치

현재 브랜치의 커밋들을 다른 브랜치의 최신 커밋 위로 옮긴다.

```bash
# feature 브랜치에서 main의 최신 커밋 위로 rebase
git switch feature/login
git rebase main

# 원격 main 기준으로 rebase
git rebase origin/main

# 충돌 발생 시
git rebase --continue     # 충돌 해결 후 계속
git rebase --skip         # 해당 커밋 건너뛰기
git rebase --abort        # rebase 취소

# 대화형 rebase (커밋 정리)
git rebase -i HEAD~3      # 최근 3개 커밋 편집
git rebase -i main        # main 이후 커밋 편집
```

### 대화형 rebase 명령어

```bash
git rebase -i HEAD~4
```

에디터에서 각 커밋 앞의 명령어를 변경한다:

| 명령어 | 축약 | 설명 |
|--------|------|------|
| `pick` | `p` | 커밋 유지 |
| `reword` | `r` | 커밋 메시지만 수정 |
| `edit` | `e` | 커밋 수정 (파일 변경 가능) |
| `squash` | `s` | 이전 커밋과 합치기 (메시지 합침) |
| `fixup` | `f` | 이전 커밋과 합치기 (메시지 버림) |
| `drop` | `d` | 커밋 삭제 |

### squash 예시 — 여러 커밋을 하나로 합치기

```bash
git rebase -i HEAD~3
```

에디터 내용:

```
pick abc1234 feat: 로그인 UI 작성
squash def5678 fix: 오타 수정
squash ghi9012 style: 들여쓰기 정리
```

→ 3개 커밋이 하나로 합쳐진다.

### fixup + autosquash — PR 커밋 자동 정리

리뷰 반영이나 오타 수정 시 `--fixup`으로 커밋하면, 나중에 `--autosquash`로 자동 정리할 수 있다. 수동 squash보다 훨씬 편리하다.

```bash
# 1. 원래 커밋이 있는 상태
git log --oneline
# a1b2c3d feat: 로그인 폼 구현
# ...

# 2. 리뷰 반영 후 fixup 커밋 생성 (대상 커밋 해시 지정)
git add .
git commit --fixup=a1b2c3d
# 자동 메시지: "fixup! feat: 로그인 폼 구현"

# 3. 메시지를 남기고 싶으면 --squash 사용
git commit --squash=a1b2c3d
# 자동 메시지: "squash! feat: 로그인 폼 구현" + 에디터에서 추가 메시지 작성

# 4. autosquash로 자동 정리
git rebase -i --autosquash main
# fixup! 커밋이 자동으로 대상 커밋 아래에 배치되고 fixup으로 표시됨
# pick a1b2c3d feat: 로그인 폼 구현
# fixup d4e5f6g fixup! feat: 로그인 폼 구현   ← 자동 배치
```

autosquash를 기본값으로 설정하면 매번 옵션을 붙이지 않아도 된다:

```bash
git config --global rebase.autosquash true
```

### --exec — 커밋마다 명령 실행

rebase 시 각 커밋 적용 후 지정한 명령을 실행한다. CI 전에 커밋별로 빌드/테스트를 검증할 때 유용하다.

```bash
# 각 커밋마다 테스트 실행
git rebase -i main --exec "npm test"

# 각 커밋마다 린트 + 빌드 검증
git rebase -i main --exec "npm run lint && npm run build"

# 실패하면 rebase가 중단되므로, 해당 커밋을 수정 후 --continue
```

### --onto — 브랜치 기반 변경

브랜치의 베이스를 다른 커밋으로 이동시킨다.

```bash
# feature가 develop에서 분기했는데, main 위로 옮기고 싶을 때
#   before: main ─ A ─ B (develop) ─ C ─ D (feature)
#   after:  main ─ A ─ C' ─ D' (feature)
git rebase --onto main develop feature/my-task

# 중간 커밋 범위 제거
# A - B - C - D - E (HEAD) 에서 C, D를 제거:
git rebase --onto B D HEAD
# 결과: A - B - E
```

---

## merge vs rebase

| 항목 | merge | rebase |
|------|-------|--------|
| 이력 | 분기 이력 보존 | 선형 이력 유지 |
| 머지 커밋 | 생성됨 (--no-ff) | 없음 |
| 안전성 | 기존 커밋 변경 없음 | 커밋 해시 변경됨 |
| 충돌 해결 | 한 번에 | 커밋마다 반복 가능 |
| 공유 브랜치 | 안전 | 위험 (이미 push한 경우) |
| 적합한 경우 | PR 병합, 공유 브랜치 | 개인 작업 브랜치 정리 |

### 실무에서의 권장 패턴

```bash
# 1. 개인 feature 브랜치를 최신화할 때 → rebase
git switch feature/my-task
git rebase origin/main

# 2. PR을 main에 병합할 때 → merge (--no-ff)
git switch main
git merge --no-ff feature/my-task

# 3. 또는 GitHub에서 "Squash and merge"로 깔끔하게
```

---

## cherry-pick — 특정 커밋만 가져오기

다른 브랜치의 특정 커밋을 현재 브랜치에 적용한다.

```bash
# 특정 커밋 가져오기
git cherry-pick <commit-hash>

# 여러 커밋 가져오기
git cherry-pick <hash1> <hash2> <hash3>

# 연속된 범위의 커밋 (시작 커밋 미포함)
git cherry-pick <start-hash>..<end-hash>

# 연속된 범위 (시작 커밋 포함)
git cherry-pick <start-hash>^..<end-hash>

# 커밋하지 않고 변경사항만 적용
git cherry-pick --no-commit <hash>
git cherry-pick -n <hash>

# 충돌 시
git cherry-pick --continue    # 충돌 해결 후 계속
git cherry-pick --abort       # 취소
git cherry-pick --skip        # 건너뛰기
```

### cherry-pick 활용 사례

```bash
# hotfix를 main과 develop 양쪽에 적용
git switch main
git cherry-pick <hotfix-commit>

git switch develop
git cherry-pick <hotfix-commit>
```

> 머지 커밋 cherry-pick, 충돌 해결, 다양한 실무 시나리오는 [15-Cherry-Pick 심화](./15-cherry-pick.md)에서 다룬다.

---

## squash merge — PR에서 커밋 합치기

GitHub에서 PR 병합 시 "Squash and merge" 옵션을 자주 사용한다. CLI에서도 가능하다.

```bash
git switch main
git merge --squash feature/login
git commit -m "feat: 로그인 기능 구현 (#42)"
```

→ feature 브랜치의 모든 커밋이 하나의 커밋으로 합쳐진다.

---

## 실무 팁

### rebase 중 꼬였을 때

```bash
# rebase 취소하고 원래 상태로
git rebase --abort

# 이미 rebase가 완료됐지만 되돌리고 싶다면
git reflog                         # 이전 상태 찾기
git reset --hard HEAD@{n}          # 해당 시점으로 복원
```

### rebase 후 push

```bash
# rebase는 커밋 해시를 변경하므로 force push 필요
git push --force-with-lease
```

### 안전 규칙

- `main`, `develop` 등 공유 브랜치에서는 **절대 rebase 하지 않는다**
- 이미 push한 커밋을 rebase했다면 `--force-with-lease`로 push한다
- 다른 팀원이 같은 브랜치에서 작업 중이면 rebase를 피한다

---

[다음: 되돌리기 →](./07-undo-and-reset.md)
