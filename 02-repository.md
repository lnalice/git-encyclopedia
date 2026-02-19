# 02. 저장소 관리

[← 목차로 돌아가기](./README.md) | [← 이전: 초기 설정](./01-initial-setup.md)

---

## 새 저장소 만들기

```bash
# 현재 디렉토리를 Git 저장소로 초기화
git init

# 새 디렉토리를 만들면서 초기화
git init my-project

# 기본 브랜치 이름 지정하며 초기화
git init -b main my-project

# bare 저장소 (서버용, 작업 디렉토리 없음)
git init --bare my-project.git
```

---

## 저장소 복제

```bash
# HTTPS로 복제
git clone https://github.com/user/repo.git

# SSH로 복제
git clone git@github.com:user/repo.git

# 특정 디렉토리 이름으로 복제
git clone https://github.com/user/repo.git my-folder

# 특정 브랜치만 복제
git clone -b develop https://github.com/user/repo.git

# 최근 커밋 1개만 복제 (얕은 복제 — CI/CD에서 유용)
git clone --depth 1 https://github.com/user/repo.git

# 특정 깊이까지 복제
git clone --depth 50 https://github.com/user/repo.git

# 서브모듈 포함 복제
git clone --recurse-submodules https://github.com/user/repo.git
```

---

## 원격 저장소 연결

```bash
# 원격 저장소 추가
git remote add origin https://github.com/user/repo.git

# 원격 저장소 목록 확인
git remote -v

# 원격 저장소 상세 정보
git remote show origin

# 원격 저장소 이름 변경
git remote rename origin upstream

# 원격 저장소 URL 변경 (HTTPS → SSH)
git remote set-url origin git@github.com:user/repo.git

# 원격 저장소 제거
git remote remove origin
```

---

## Fork한 저장소 관리

오픈소스 기여 시 원본 저장소를 upstream으로 등록한다.

```bash
# 1. Fork한 저장소 복제
git clone git@github.com:my-account/forked-repo.git

# 2. 원본 저장소를 upstream으로 등록
git remote add upstream https://github.com/original-owner/repo.git

# 3. 확인
git remote -v
# origin    git@github.com:my-account/forked-repo.git (fetch)
# origin    git@github.com:my-account/forked-repo.git (push)
# upstream  https://github.com/original-owner/repo.git (fetch)
# upstream  https://github.com/original-owner/repo.git (push)

# 4. 원본 최신 코드 동기화
git fetch upstream
git checkout main
git merge upstream/main
```

---

## sparse-checkout — 부분 체크아웃

대규모 모노레포에서 필요한 디렉토리만 체크아웃하여 디스크와 시간을 절약한다.

```bash
# sparse-checkout으로 clone
git clone --no-checkout https://github.com/team/monorepo.git
cd monorepo
git sparse-checkout init --cone

# 필요한 디렉토리만 지정
git sparse-checkout set packages/auth packages/common
git checkout main

# 디렉토리 추가
git sparse-checkout add packages/payment

# 현재 설정 확인
git sparse-checkout list

# 전체 체크아웃으로 복원
git sparse-checkout disable
```

### cone 모드 vs non-cone 모드

```bash
# cone 모드 (권장, Git 2.25+) — 디렉토리 단위
git sparse-checkout init --cone
git sparse-checkout set src/frontend src/shared

# non-cone 모드 — 패턴으로 세밀하게 지정
git sparse-checkout init --no-cone
git sparse-checkout set '*.md' 'src/auth/**' 'configs/*'
```

### 부분 clone과 조합

```bash
# blob 없이 clone + sparse-checkout (가장 빠른 초기 설정)
git clone --filter=blob:none --sparse https://github.com/team/monorepo.git
cd monorepo
git sparse-checkout set packages/my-service
```

---

## 저장소 정보 확인

```bash
# 현재 저장소 상태
git status

# 저장소 루트 디렉토리 경로
git rev-parse --show-toplevel

# .git 디렉토리 위치
git rev-parse --git-dir

# 현재 브랜치 이름
git branch --show-current

# 원격 저장소 URL만 빠르게 확인
git remote get-url origin
```

---

## 저장소 정리

```bash
# 불필요한 객체 정리 (가비지 컬렉션)
git gc

# 적극적 정리 (저장소 크기 최적화)
git gc --aggressive

# 원격에서 삭제된 브랜치 참조 정리
git remote prune origin

# 또는 fetch 시 자동 정리
git fetch --prune
```

---

## 실무 팁

### GitHub에서 새 프로젝트 시작하는 일반적인 흐름

```bash
# GitHub에서 저장소 생성 후
git clone git@github.com:my-team/new-project.git
cd new-project

# 초기 파일 생성
echo "# New Project" > README.md
git add README.md
git commit -m "Initial commit"
git push -u origin main
```

### 기존 로컬 프로젝트를 GitHub에 올리기

```bash
cd existing-project
git init
git add .
git commit -m "Initial commit"

# GitHub에서 빈 저장소 생성 후
git remote add origin git@github.com:user/repo.git
git push -u origin main
```

---

[다음: 스테이징 & 커밋 →](./03-staging-and-commit.md)
