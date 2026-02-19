# 14. .gitignore & 파일 관리

[← 목차로 돌아가기](./README.md) | [← 이전: GitHub CLI](./13-github-cli.md)

---

## .gitignore란?

Git이 추적하지 않을 파일과 디렉토리를 지정하는 설정 파일이다. 프로젝트 루트에 `.gitignore` 파일을 생성한다.

---

## 기본 문법

```gitignore
# 주석
# '#'으로 시작하는 줄은 무시된다

# 특정 파일
secret.key
config.local.json

# 특정 확장자
*.log
*.tmp
*.swp

# 특정 디렉토리 (끝에 / 필수)
node_modules/
dist/
build/
.cache/

# 와일드카드
*.py[cod]          # .pyc, .pyo, .pyd
doc/**/*.pdf       # doc 하위 모든 pdf

# 부정 패턴 (예외)
*.log
!important.log     # important.log는 추적

# 루트 디렉토리 기준 (앞에 / 붙이기)
/TODO              # 루트의 TODO만 무시 (하위 디렉토리의 TODO는 추적)
```

### 패턴 규칙 정리

| 패턴 | 설명 |
|------|------|
| `file.txt` | 모든 위치의 file.txt |
| `/file.txt` | 루트의 file.txt만 |
| `dir/` | 모든 위치의 dir 디렉토리 |
| `/dir/` | 루트의 dir 디렉토리만 |
| `*.log` | 모든 .log 파일 |
| `!keep.log` | keep.log는 예외 (추적) |
| `**/*.txt` | 모든 하위의 .txt 파일 |
| `doc/**` | doc 디렉토리 하위 전체 |

---

## 언어/프레임워크별 템플릿

### Node.js / JavaScript

```gitignore
node_modules/
dist/
build/
.env
.env.local
.env.*.local
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.eslintcache
.parcel-cache/
.next/
.nuxt/
coverage/
```

### Python

```gitignore
__pycache__/
*.py[cod]
*$py.class
*.egg-info/
dist/
build/
.eggs/
*.egg
.venv/
venv/
env/
.env
.pytest_cache/
.mypy_cache/
htmlcov/
.coverage
```

### Java / Kotlin

```gitignore
*.class
*.jar
*.war
*.ear
target/
build/
.gradle/
out/
.idea/
*.iml
```

### 공통 (OS & 에디터)

```gitignore
# macOS
.DS_Store
.AppleDouble
.LSOverride
._*

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini

# Linux
*~
.directory

# VS Code
.vscode/
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json

# JetBrains (IntelliJ, WebStorm 등)
.idea/
*.iml

# Vim
*.swp
*.swo
*~
```

---

## 이미 추적 중인 파일 무시하기

`.gitignore`에 추가해도 **이미 추적 중인 파일**은 계속 추적된다. 추적을 중지하려면:

```bash
# 특정 파일 추적 중지 (파일은 디스크에 유지)
git rm --cached file.txt

# 디렉토리 추적 중지
git rm -r --cached directory/

# .gitignore 수정 후 전체 캐시 재설정
git rm -r --cached .
git add .
git commit -m "chore: .gitignore 재적용"
```

---

## 추적 파일의 로컬 변경 무시 (assume-unchanged / skip-worktree)

`.gitignore`는 **추적되지 않는** 파일용이다. 이미 추적 중인 파일의 로컬 변경을 Git이 무시하게 하려면 `update-index`를 사용한다.

### --assume-unchanged

Git이 해당 파일의 변경 여부를 확인하지 않도록 한다. 성능 최적화 목적으로 설계되었지만, 로컬 설정 파일 용도로도 쓰인다.

```bash
# 로컬 변경을 무시하도록 설정
git update-index --assume-unchanged config/database.yml

# 다시 추적
git update-index --no-assume-unchanged config/database.yml

# 현재 설정된 파일 목록 확인
git ls-files -v | grep '^h'
```

### --skip-worktree (권장)

사용자가 의도적으로 변경한 파일을 무시할 때 사용한다. `--assume-unchanged`보다 의미적으로 적합하다.

```bash
# 로컬 변경 무시 설정
git update-index --skip-worktree config/local-settings.json

# 해제
git update-index --no-skip-worktree config/local-settings.json

# 현재 설정된 파일 목록 확인
git ls-files -v | grep '^S'
```

### 차이점 정리

| 항목 | assume-unchanged | skip-worktree |
|------|-----------------|---------------|
| 목적 | 성능 최적화 (대형 파일) | 로컬 전용 변경 보호 |
| `git reset --hard` 시 | 덮어쓰일 수 있음 | 보호됨 |
| `git pull` 충돌 시 | 무시될 수 있음 | 경고 표시 |
| 권장 용도 | 변경하지 않는 대형 파일 | 로컬 DB 설정, API 키 등 |

### 실무 활용

```bash
# 로컬 DB 접속 정보를 개인 환경에 맞게 수정한 후 커밋 방지
git update-index --skip-worktree config/database.yml

# 팀원에게 알려주기: README에 기록
# "config/database.yml을 로컬 환경에 맞게 수정 후
#  git update-index --skip-worktree config/database.yml 실행"
```

> **주의**: 이 설정은 **로컬 전용**이다. 팀원 각자 설정해야 하며, 근본적으로는 `.env` + `.env.example` 패턴이 더 바람직하다.

---

## 전역 .gitignore

모든 프로젝트에 공통으로 적용할 무시 규칙을 설정한다.

```bash
# 전역 gitignore 파일 생성
git config --global core.excludesFile ~/.gitignore_global
```

`~/.gitignore_global` 내용:

```gitignore
.DS_Store
Thumbs.db
*.swp
*~
.idea/
.vscode/
```

---

## .gitignore 확인 & 디버깅

```bash
# 특정 파일이 왜 무시되는지 확인
git check-ignore -v file.txt
# .gitignore:3:*.log    file.log

# 무시되는 파일 전체 목록
git status --ignored

# 무시되는 파일을 강제로 추가
git add -f ignored-file.txt
```

---

## .gitkeep — 빈 디렉토리 유지

Git은 빈 디렉토리를 추적하지 않는다. 빈 디렉토리를 유지하려면 `.gitkeep` 파일을 넣는다.

```bash
# 빈 디렉토리 유지
mkdir logs
touch logs/.gitkeep
git add logs/.gitkeep

# .gitignore와 함께 사용
# 디렉토리 내 파일은 무시하되, 디렉토리 자체는 유지
logs/*
!logs/.gitkeep
```

---

## .gitattributes — 파일 속성 설정

줄바꿈, diff 방식, merge 전략 등을 파일별로 지정한다.

```gitattributes
# 줄바꿈 자동 변환
* text=auto

# 특정 파일 줄바꿈 강제
*.sh text eol=lf
*.bat text eol=crlf

# 바이너리 파일 (diff/merge 하지 않음)
*.png binary
*.jpg binary
*.pdf binary

# Git LFS 파일
*.psd filter=lfs diff=lfs merge=lfs -text

# 특정 파일 diff 비활성화
package-lock.json -diff
yarn.lock -diff

# 언어별 diff 드라이버
*.py diff=python
*.rb diff=ruby
```

---

## Git LFS — 대용량 파일 관리

Git은 대용량 바이너리 파일(이미지, 동영상, 디자인 파일 등)에 적합하지 않다. Git LFS는 이런 파일을 별도 서버에 저장하고, 저장소에는 포인터만 남긴다.

```bash
# Git LFS 설치
brew install git-lfs       # macOS
# apt install git-lfs      # Ubuntu
git lfs install

# 추적 패턴 등록
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "assets/**/*.png"

# .gitattributes에 자동 기록됨 — 반드시 커밋
git add .gitattributes
git commit -m "chore: Git LFS 설정"

# 이후 해당 파일을 정상적으로 add/commit/push
git add design.psd
git commit -m "feat: 디자인 파일 추가"
git push

# LFS로 관리 중인 파일 확인
git lfs ls-files

# LFS 상태 확인
git lfs status
```

### LFS 관련 주의사항

```bash
# clone 시 LFS 파일도 함께 다운로드 (기본 동작)
git clone https://github.com/team/repo.git

# LFS 파일 없이 clone (CI에서 빌드 불필요 시)
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/team/repo.git

# 나중에 LFS 파일 다운로드
git lfs pull
```

---

## 실무 팁

### 프로젝트 시작 시

```bash
# GitHub의 .gitignore 템플릿 활용
# https://github.com/github/gitignore

# gh CLI로 저장소 생성 시 템플릿 지정
gh repo create my-project --gitignore Node
```

### .env 파일 관리

```gitignore
# 실제 환경변수는 무시
.env
.env.local
.env.production

# 템플릿은 추적 (어떤 변수가 필요한지 문서화)
!.env.example
```

`.env.example` 예시:

```
DATABASE_URL=postgresql://user:password@localhost:5432/db
JWT_SECRET=your-secret-key-here
API_KEY=your-api-key
```

### 프로젝트 중간에 .gitignore 추가

```bash
# 1. .gitignore 파일 생성/수정
# 2. 캐시 초기화
git rm -r --cached .
# 3. 다시 추가
git add .
# 4. 커밋
git commit -m "chore: .gitignore 적용"
```

---

[다음: Cherry-Pick 심화 →](./15-cherry-pick.md) | [← 목차로 돌아가기](./README.md)
