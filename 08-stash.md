# 08. 임시 저장 (Stash)

[← 목차로 돌아가기](./README.md) | [← 이전: 되돌리기](./07-undo-and-reset.md)

---

## stash란?

작업 중인 변경사항을 임시로 저장하고, 워킹 디렉토리를 깨끗하게 만들 수 있다. 급한 작업이 들어왔을 때 현재 작업을 잠시 치워두는 용도로 사용한다.

---

## 기본 사용법

```bash
# 현재 변경사항 임시 저장
git stash

# 메시지와 함께 저장
git stash push -m "로그인 폼 작업 중"

# 특정 파일만 stash
git stash push -m "부분 저장" -- file1.txt file2.txt

# 추적되지 않는 파일(untracked)도 포함
git stash -u
git stash --include-untracked

# .gitignore 파일까지 모두 포함
git stash -a
git stash --all
```

---

## stash 목록 관리

```bash
# stash 목록 보기
git stash list
# stash@{0}: On feature/login: 로그인 폼 작업 중
# stash@{1}: WIP on main: abc1234 이전 작업

# 특정 stash 내용 확인
git stash show              # 변경 파일 목록
git stash show -p           # diff 포함
git stash show stash@{1}    # 특정 stash 확인
git stash show -p stash@{1}
```

---

## stash 복원

```bash
# 가장 최근 stash 복원 (stash 유지)
git stash apply

# 가장 최근 stash 복원 + 삭제
git stash pop

# 특정 stash 복원
git stash apply stash@{2}
git stash pop stash@{2}

# 스테이징 상태까지 복원 (기본: 모두 unstaged로 복원)
git stash apply --index
git stash pop --index
```

### apply vs pop

| 명령어 | stash 목록에서 | 사용 시기 |
|--------|---------------|-----------|
| `apply` | 유지 | 여러 브랜치에 같은 변경 적용할 때 |
| `pop` | 삭제 | 일반적인 복원 |

---

## stash 삭제

```bash
# 특정 stash 삭제
git stash drop stash@{0}

# 모든 stash 삭제
git stash clear
```

---

## stash로 브랜치 만들기

stash를 복원할 때 충돌이 발생하면, 새 브랜치를 만들어서 적용할 수 있다.

```bash
# stash 내용으로 새 브랜치 생성 + 복원
git stash branch new-branch-name

# 특정 stash로
git stash branch new-branch stash@{2}
```

---

## 부분 stash (패치 모드)

변경사항 중 일부만 선택적으로 stash할 수 있다.

```bash
git stash push -p
# 각 hunk마다 y/n으로 선택
# y: stash에 포함
# n: 워킹 디렉토리에 유지
# s: 더 작게 분할
# q: 종료
```

---

## 실무 시나리오

### 급한 버그 수정 요청이 들어왔을 때

```bash
# 1. 현재 작업 임시 저장
git stash push -m "feature/payment 작업 중"

# 2. 핫픽스 브랜치로 이동
git switch main
git pull
git switch -c hotfix/critical-bug

# 3. 버그 수정 & 커밋 & push
# ...
git push -u origin hotfix/critical-bug

# 4. 원래 작업으로 복귀
git switch feature/payment
git stash pop
```

### 브랜치를 잘못 잡고 작업했을 때

```bash
# 1. 변경사항 stash
git stash

# 2. 올바른 브랜치로 이동
git switch feature/correct-branch

# 3. stash 복원
git stash pop
```

### 실험적 변경사항을 보존하면서 원래 상태로 돌아가고 싶을 때

```bash
# 실험 코드 stash (apply는 삭제하지 않으므로)
git stash push -m "실험: 새 알고리즘 테스트"

# 나중에 필요하면 apply로 복원
git stash apply stash@{n}
```

---

[다음: 로그 & 히스토리 →](./09-log-and-history.md)
