# 07. 되돌리기 (Undo & Reset)

[← 목차로 돌아가기](./README.md) | [← 이전: 병합 & 리베이스](./06-merge-and-rebase.md)

---

## 되돌리기 명령어 한눈에 보기

| 상황 | 명령어 | 안전성 |
|------|--------|--------|
| 워킹 디렉토리 변경 취소 | `git restore <file>` | 위험 (복구 불가) |
| 스테이징 취소 | `git restore --staged <file>` | 안전 |
| 마지막 커밋 메시지 수정 | `git commit --amend` | push 전만 |
| 특정 커밋 되돌리기 (새 커밋) | `git revert <hash>` | 안전 |
| 커밋 이력 자체를 되돌리기 | `git reset` | 위험도 다양 |
| 삭제된 커밋 복구 | `git reflog` + `reset` | 안전 |

---

## restore — 파일 수준 되돌리기

```bash
# 워킹 디렉토리 변경 취소 (마지막 커밋 상태로)
git restore file.txt
git restore .                        # 모든 파일

# 스테이징 취소 (워킹 디렉토리 변경은 유지)
git restore --staged file.txt
git restore --staged .

# 특정 커밋 시점으로 파일 복원
git restore --source=abc1234 file.txt

# 스테이징 + 워킹 디렉토리 모두 되돌리기
git restore --staged --worktree file.txt
```

---

## revert — 커밋 되돌리기 (안전)

기존 커밋을 취소하는 **새 커밋**을 만든다. 이력이 보존되므로 공유 브랜치에서도 안전하다.

```bash
# 특정 커밋 되돌리기
git revert <commit-hash>

# 메시지 에디터 없이 바로 커밋
git revert --no-edit <commit-hash>

# 커밋하지 않고 변경사항만 적용
git revert --no-commit <commit-hash>
git revert -n <commit-hash>

# 여러 커밋 연속 되돌리기
git revert <older-hash>..<newer-hash>

# 머지 커밋 되돌리기 (-m 1: 첫 번째 부모 기준)
git revert -m 1 <merge-commit-hash>

# revert 중단
git revert --abort
```

### revert 사용 시나리오

```bash
# 배포 후 버그 발견 → 해당 커밋만 되돌리기
git log --oneline              # 문제 커밋 해시 확인
git revert abc1234
git push origin main           # 안전하게 push
```

---

## reset — 커밋 이력 되돌리기

HEAD의 위치를 과거 커밋으로 이동시킨다. **3가지 모드**가 있다.

### --soft

커밋만 취소. 변경사항은 스테이징 영역에 유지.

```bash
git reset --soft HEAD~1

# 활용: 마지막 커밋을 다시 수정해서 커밋하고 싶을 때
git reset --soft HEAD~1
# (파일 수정)
git commit -m "수정된 커밋 메시지"
```

### --mixed (기본값)

커밋 + 스테이징 취소. 변경사항은 워킹 디렉토리에 유지.

```bash
git reset HEAD~1
git reset --mixed HEAD~1       # 동일

# 활용: 커밋을 취소하고 다시 선별적으로 스테이징하고 싶을 때
git reset HEAD~2
git add file1.txt
git commit -m "첫 번째 변경"
git add file2.txt
git commit -m "두 번째 변경"
```

### --hard

커밋 + 스테이징 + 워킹 디렉토리 모두 삭제. **복구 어려움**.

```bash
git reset --hard HEAD~1

# 특정 커밋으로 완전 복귀
git reset --hard abc1234

# 원격과 동일하게 맞추기
git reset --hard origin/main
```

> **주의**: `--hard`는 워킹 디렉토리의 변경사항을 완전히 삭제한다. 커밋되지 않은 작업이 영구적으로 사라질 수 있다.

### reset 모드 비교

```
                    커밋    스테이징    워킹 디렉토리
--soft              취소    유지        유지
--mixed (기본)      취소    취소        유지
--hard              취소    취소        삭제
```

### 특정 파일만 reset

```bash
# 특정 파일의 스테이징만 취소 (git restore --staged와 동일)
git reset HEAD file.txt

# 특정 커밋 시점의 파일로 스테이징
git reset <commit-hash> -- file.txt
```

---

## reflog — 잃어버린 커밋 복구

`reflog`는 HEAD가 가리켰던 모든 커밋의 기록을 보여준다. 실수로 삭제한 커밋도 찾을 수 있다.

```bash
# HEAD 이동 이력 보기
git reflog

# 출력 예시:
# abc1234 HEAD@{0}: reset: moving to HEAD~3
# def5678 HEAD@{1}: commit: feat: 로그인 구현
# ghi9012 HEAD@{2}: commit: fix: 버그 수정
# jkl3456 HEAD@{3}: checkout: moving from main to feature

# 특정 시점으로 복구
git reset --hard HEAD@{1}

# 또는 특정 해시로 복구
git reset --hard def5678

# 삭제된 브랜치 복구
git reflog
# mno7890 HEAD@{5}: checkout: moving from deleted-branch to main
git checkout -b recovered-branch mno7890
```

> **안전망**: `git reset --hard`를 실수로 했어도, 30일 이내라면 `reflog`로 복구할 수 있다.

---

## clean — 추적되지 않는 파일 삭제

```bash
# 삭제 대상 미리 보기 (dry run)
git clean -n

# 추적되지 않는 파일 삭제
git clean -f

# 디렉토리도 포함
git clean -fd

# .gitignore에 포함된 파일도 삭제
git clean -fX

# 모든 추적되지 않는 파일 삭제 (gitignore 포함)
git clean -fdx

# 대화형 모드
git clean -i
```

---

## filter-repo — 이력 재작성

전체 커밋 이력에서 특정 파일을 제거하거나, 작성자 정보를 일괄 변경한다. `git filter-branch`의 후속 도구로, 별도 설치가 필요하다.

```bash
# 설치
pip install git-filter-repo

# 모든 커밋에서 특정 파일 제거
git filter-repo --invert-paths --path secrets.env

# 디렉토리를 루트로 추출 (모노레포 분리 시)
git filter-repo --subdirectory-filter src/

# 작성자 이메일 일괄 변경
git filter-repo --email-callback '
    return email.replace(b"old@email.com", b"new@email.com")
'

# 대용량 파일 제거
git filter-repo --strip-blobs-bigger-than 10M
```

> **주의**: 이력 재작성은 모든 커밋 해시를 변경한다. 팀원 전체와 합의 후 진행하고, 완료 후 모든 팀원이 fresh clone해야 한다.

### 실수로 커밋한 민감 정보 제거

```bash
# 1. filter-repo로 해당 파일 이력 완전 삭제
git filter-repo --invert-paths --path .env

# 2. 원격에 force push
git push --force --all

# 3. 팀원 전체 fresh clone 필요
# 4. GitHub에서 해당 시크릿 즉시 교체 (유출된 키는 무효화)
```

---

## 실무 시나리오별 가이드

### "커밋 메시지를 잘못 썼어요" (push 전)

```bash
git commit --amend -m "올바른 메시지"
```

### "파일을 빠뜨리고 커밋했어요" (push 전)

```bash
git add forgotten-file.txt
git commit --amend --no-edit
```

### "방금 한 커밋을 취소하고 싶어요" (push 전)

```bash
git reset --soft HEAD~1     # 변경사항은 스테이징에 유지
```

### "이미 push한 커밋을 되돌리고 싶어요"

```bash
git revert <commit-hash>    # 새 커밋으로 안전하게 되돌리기
git push
```

### "로컬을 원격과 완전히 동일하게 맞추고 싶어요"

```bash
git fetch origin
git reset --hard origin/main
git clean -fd               # 추적되지 않는 파일도 정리
```

### "reset --hard를 실수로 했어요"

```bash
git reflog                  # 이전 커밋 해시 찾기
git reset --hard HEAD@{n}   # 복구
```

---

[다음: 임시 저장 (Stash) →](./08-stash.md)
