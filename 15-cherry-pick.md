# 15. Cherry-Pick 심화

[← 목차로 돌아가기](./README.md) | [← 이전: .gitignore & 파일 관리](./14-gitignore.md)

---

## cherry-pick이란?

다른 브랜치의 **특정 커밋**만 골라서 현재 브랜치에 적용하는 명령어이다. 전체 브랜치를 merge하지 않고, 원하는 변경사항만 가져올 수 있다.

```
develop:  A ─ B ─ C ─ D ─ E
                      ↓ cherry-pick
main:     X ─ Y ─ Z ─ D'
```

> `D'`는 `D`와 동일한 변경 내용이지만 **다른 커밋 해시**를 갖는다.

---

## 기본 사용법

```bash
# 단일 커밋 가져오기
git cherry-pick <commit-hash>

# 여러 커밋 가져오기 (각각 별도 커밋으로)
git cherry-pick <hash1> <hash2> <hash3>

# 연속된 범위 — 시작 커밋 미포함, 끝 커밋 포함
git cherry-pick A..C        # B, C를 가져옴 (A 제외)

# 연속된 범위 — 시작 커밋 포함
git cherry-pick A^..C       # A, B, C 모두 가져옴
```

---

## 주요 옵션

### --no-commit (-n)

커밋하지 않고 변경사항만 워킹 디렉토리와 스테이징 영역에 적용한다. 여러 커밋을 하나로 합칠 때 유용하다.

```bash
git cherry-pick --no-commit <hash1>
git cherry-pick --no-commit <hash2>
git cherry-pick --no-commit <hash3>

# 세 커밋의 변경사항을 하나의 커밋으로
git commit -m "feat: 여러 변경사항 통합 적용"
```

### --edit (-e)

cherry-pick 시 커밋 메시지를 에디터에서 수정할 수 있다.

```bash
git cherry-pick --edit <hash>
```

### --signoff (-s)

커밋 메시지에 `Signed-off-by` 라인을 추가한다. 오픈소스 프로젝트에서 요구하는 경우가 많다.

```bash
git cherry-pick --signoff <hash>
# 커밋 메시지 하단에 추가됨:
# Signed-off-by: 홍길동 <gildong@example.com>
```

### -x

커밋 메시지에 원본 커밋 해시를 자동으로 기록한다. 어디서 가져왔는지 추적할 때 유용하다.

```bash
git cherry-pick -x <hash>
# 커밋 메시지 하단에 추가됨:
# (cherry picked from commit abc1234567890...)
```

> **실무 권장**: 다른 브랜치에서 cherry-pick할 때는 항상 `-x`를 붙여서 출처를 남기는 것이 좋다.

### --strategy / --strategy-option

병합 전략을 지정한다.

```bash
# 충돌 시 현재 브랜치(ours) 우선
git cherry-pick --strategy-option=ours <hash>

# 충돌 시 가져오는 커밋(theirs) 우선
git cherry-pick --strategy-option=theirs <hash>
```

---

## 머지 커밋 cherry-pick

머지 커밋은 부모가 2개이므로 어느 부모 기준으로 diff를 계산할지 `-m` 옵션으로 지정해야 한다.

```bash
# 머지 커밋 확인
git log --oneline --graph
# *   M  Merge branch 'feature' into main
# |\
# | * C  feat: 기능 구현
# * | B  fix: 버그 수정
# |/
# * A  이전 커밋

# -m 1: 첫 번째 부모(main) 기준 diff → feature 브랜치의 변경만 적용
git cherry-pick -m 1 <merge-commit-hash>

# -m 2: 두 번째 부모(feature) 기준 diff → main 브랜치의 변경만 적용
git cherry-pick -m 2 <merge-commit-hash>
```

### 부모 번호 확인

```bash
# 머지 커밋의 부모 확인
git cat-file -p <merge-commit-hash>
# tree ...
# parent abc1234   ← 1번 부모 (보통 merge를 실행한 브랜치)
# parent def5678   ← 2번 부모 (merge된 브랜치)
```

> **일반적으로 `-m 1`을 사용**한다. 이렇게 하면 merge된 브랜치(feature)의 변경사항이 적용된다.

---

## 충돌 해결

### 충돌 발생 시 흐름

```bash
git cherry-pick <hash>
# CONFLICT (content): Merge conflict in src/app.js

# 1. 충돌 파일 확인
git status

# 2. 충돌 해결 (에디터에서 파일 수정)

# 3. 해결된 파일 스테이징
git add src/app.js

# 4. cherry-pick 계속
git cherry-pick --continue
```

### 연속 cherry-pick 중 충돌

여러 커밋을 cherry-pick할 때, 중간에 충돌이 나면 하나씩 해결하며 진행한다.

```bash
git cherry-pick A^..E
# A 적용 완료
# B 적용 완료
# C에서 CONFLICT 발생

# 충돌 해결 후
git add .
git cherry-pick --continue
# D 적용 완료
# E 적용 완료
```

### 중단 & 건너뛰기

```bash
# cherry-pick 전체 취소 (시작 전 상태로 복귀)
git cherry-pick --abort

# 현재 커밋 건너뛰고 다음으로 진행
git cherry-pick --skip

# cherry-pick 중단하되, 지금까지 적용된 건 유지
git cherry-pick --quit
```

---

## 실무 시나리오

### 1. 핫픽스를 여러 브랜치에 적용

프로덕션 버그를 수정한 커밋을 main, develop, release 브랜치에 모두 적용해야 할 때.

```bash
# main에서 핫픽스 커밋 생성
git switch main
git commit -m "fix: 결제 금액 소수점 오류 수정"
# → 커밋 해시: a1b2c3d

# develop에 적용
git switch develop
git cherry-pick -x a1b2c3d

# release 브랜치에도 적용
git switch release/v2.1
git cherry-pick -x a1b2c3d
```

### 2. 다른 팀원의 특정 커밋만 가져오기

팀원이 작업 중인 브랜치에서 필요한 유틸 함수 커밋만 먼저 가져올 때.

```bash
# 팀원 브랜치의 커밋 확인
git fetch origin
git log origin/feature/teammate-work --oneline

# 필요한 커밋만 선택
git cherry-pick -x <유틸함수-커밋-hash>
```

### 3. 잘못된 브랜치에 한 커밋을 올바른 브랜치로 이동

```bash
# 현재 feature/wrong 브랜치에 커밋이 들어간 상태
git log --oneline -3
# a1b2c3d feat: 새 기능     ← 이 커밋을 옮겨야 함
# ...

# 올바른 브랜치에서 cherry-pick
git switch feature/correct
git cherry-pick a1b2c3d

# 잘못된 브랜치에서 커밋 제거
git switch feature/wrong
git reset --hard HEAD~1
```

### 4. 릴리스 브랜치에 특정 기능만 포함

develop에 여러 기능이 있지만, 이번 릴리스에는 일부만 포함할 때.

```bash
git switch -c release/v3.0 main

# 포함할 기능 커밋들만 선택적으로 적용
git cherry-pick -x <로그인-개선-hash>
git cherry-pick -x <검색-최적화-hash>
# 결제 리팩토링은 다음 릴리스로 미룸
```

### 5. 다른 Fork/원격 저장소에서 커밋 가져오기

오픈소스에서 다른 기여자의 패치를 가져올 때.

```bash
# 상대방의 저장소를 원격으로 추가
git remote add contributor https://github.com/contributor/repo.git
git fetch contributor

# 원하는 커밋 cherry-pick
git cherry-pick <contributor-commit-hash>

# 정리
git remote remove contributor
```

### 6. revert한 커밋을 다시 적용

실수로 revert한 기능을 다시 살릴 때, 원본 커밋을 cherry-pick한다.

```bash
# revert 커밋이 있는 상태
git log --oneline
# r1r2r3r Revert "feat: 새 기능"
# a1b2c3d feat: 새 기능

# 원본 커밋을 다시 적용
git cherry-pick a1b2c3d
```

---

## cherry-pick vs 다른 명령어

| 상황 | 추천 명령어 | 이유 |
|------|------------|------|
| 브랜치 전체를 합칠 때 | `merge` | 전체 이력 보존 |
| 브랜치를 정리하면서 합칠 때 | `rebase` | 선형 이력 유지 |
| 특정 커밋 1~2개만 필요할 때 | `cherry-pick` | 선택적 적용 |
| 특정 커밋을 되돌릴 때 | `revert` | 안전한 취소 |

---

## 주의사항

### 중복 커밋 문제

같은 변경을 cherry-pick과 merge로 두 번 적용하면 충돌이 발생할 수 있다.

```bash
# develop에서 cherry-pick으로 가져온 커밋이 있는 상태에서
# develop 전체를 merge하면 → 같은 변경이 두 번 적용되며 충돌 가능

# 해결 방법: cherry-pick 대신 merge를 사용하거나,
# merge 전에 cherry-pick한 커밋을 revert 후 merge
```

### 커밋 해시 변경

cherry-pick은 새로운 커밋을 만들기 때문에, 원본과 해시가 다르다. `-x` 옵션으로 출처를 기록하거나 PR에 명시해두는 것이 좋다.

### cherry-pick 남용 금지

cherry-pick을 지나치게 많이 사용하면 이력 추적이 어려워진다. 브랜치 전체를 가져올 수 있다면 merge나 rebase를 사용하는 것이 바람직하다.

---

[다음: Submodule & Subtree →](./16-submodule-and-subtree.md)
