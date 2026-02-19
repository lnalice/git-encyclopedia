# 05. 원격 저장소

[← 목차로 돌아가기](./README.md) | [← 이전: 브랜치 관리](./04-branch.md)

---

## fetch — 원격 변경사항 가져오기

원격 저장소의 최신 정보를 다운로드하지만, 로컬 브랜치에 **자동으로 병합하지 않는다**.

```bash
# 기본 원격(origin)에서 가져오기
git fetch

# 특정 원격에서 가져오기
git fetch upstream

# 특정 브랜치만 가져오기
git fetch origin main

# 모든 원격에서 가져오기
git fetch --all

# 삭제된 원격 브랜치 참조도 정리
git fetch --prune

# 태그도 함께 가져오기
git fetch --tags
```

### fetch 후 확인

```bash
# 원격과 로컬 차이 확인
git log HEAD..origin/main --oneline

# 변경된 파일 확인
git diff HEAD..origin/main --stat
```

---

## pull — 가져오기 + 병합

`fetch` + `merge`를 한 번에 수행한다.

```bash
# 현재 브랜치의 추적 원격 브랜치에서 pull
git pull

# 특정 원격/브랜치에서 pull
git pull origin main

# rebase 방식으로 pull (불필요한 머지 커밋 방지)
git pull --rebase
git pull --rebase origin main

# pull 기본 전략을 rebase로 설정
git config --global pull.rebase true

# fast-forward만 허용 (충돌 시 중단)
git pull --ff-only

# 모든 원격에서 pull
git pull --all
```

### pull vs fetch + merge

```bash
# 아래 두 줄은 동일하다
git pull origin main

git fetch origin main
git merge origin/main
```

> **실무 권장**: 충돌 가능성이 있다면 `fetch` → 확인 → `merge/rebase` 순서가 안전하다.

---

## push — 원격에 업로드

```bash
# 현재 브랜치를 추적 원격 브랜치에 push
git push

# 특정 원격/브랜치에 push
git push origin main

# 새 브랜치를 원격에 push + 추적 설정
git push -u origin feature/login
git push --set-upstream origin feature/login

# 모든 브랜치 push
git push --all

# 태그 push
git push origin v1.0.0
git push --tags              # 모든 태그

# 강제 push (주의: 원격 이력 덮어쓰기)
git push --force
git push -f

# 안전한 강제 push (다른 사람의 커밋이 없을 때만)
git push --force-with-lease

# 원격 브랜치 삭제
git push origin --delete feature/old-branch
```

> **절대 주의**: `--force`는 원격 이력을 덮어쓴다. `main`/`develop` 브랜치에는 절대 사용하지 않는다. 불가피한 경우 `--force-with-lease`를 사용한다.

---

## force-with-lease 상세

`--force-with-lease`는 마지막으로 fetch한 이후 다른 사람이 push한 커밋이 없을 때만 force push를 허용한다.

```bash
# rebase 후 안전한 강제 push
git rebase origin/main
git push --force-with-lease

# 특정 브랜치에 대해
git push --force-with-lease origin feature/my-branch
```

### --force-if-includes (Git 2.30+)

`--force-with-lease`는 백그라운드에서 `git fetch`가 실행된 경우 의도치 않게 다른 사람의 커밋을 덮어쓸 수 있다. `--force-if-includes`는 로컬에서 실제로 확인한 원격 커밋을 기준으로 검사하므로 더 안전하다.

```bash
# 가장 안전한 강제 push
git push --force-with-lease --force-if-includes

# alias로 등록해두면 편리
git config --global alias.fpush "push --force-with-lease --force-if-includes"
git fpush origin feature/my-branch
```

---

## 원격 브랜치 관리

```bash
# 원격 브랜치 목록
git branch -r

# 원격 브랜치 상세 정보
git remote show origin

# 원격에서 삭제된 브랜치 로컬 참조 정리
git fetch --prune

# 자동 prune 설정
git config --global fetch.prune true

# 원격 브랜치를 로컬로 체크아웃
git checkout -b local-name origin/remote-name
git switch -c local-name origin/remote-name
```

---

## 여러 원격 저장소 사용

오픈소스 기여나 여러 환경 배포 시 복수의 원격을 사용한다.

```bash
# 원격 추가
git remote add staging git@deploy-server:repo.git
git remote add production git@production:repo.git

# 특정 원격에 push
git push staging develop
git push production main

# 원격 목록 확인
git remote -v
# origin      git@github.com:team/repo.git (fetch)
# origin      git@github.com:team/repo.git (push)
# staging     git@deploy-server:repo.git (fetch)
# staging     git@deploy-server:repo.git (push)
# production  git@production:repo.git (fetch)
# production  git@production:repo.git (push)
```

---

## 실무 팁

### push 전 확인 습관

```bash
# push할 커밋 미리 확인
git log origin/main..HEAD --oneline

# push할 변경사항 확인
git diff origin/main..HEAD --stat
```

### pull 충돌 최소화

```bash
# 작업 시작 전 항상
git pull --rebase origin main

# 장기 작업 중 주기적으로
git fetch origin
git rebase origin/main
```

### 실수로 push한 커밋 되돌리기

```bash
# revert로 되돌리기 (안전 — 새 커밋 생성)
git revert <commit-hash>
git push

# force push로 되돌리기 (위험 — 이력 제거)
git reset --hard <이전-commit-hash>
git push --force-with-lease
```

---

[다음: 병합 & 리베이스 →](./06-merge-and-rebase.md)
