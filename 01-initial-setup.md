# 01. Git 초기 설정

[← 목차로 돌아가기](./README.md)

---

## 사용자 정보 설정

Git 설치 후 가장 먼저 해야 하는 설정. 커밋 기록에 남는 이름과 이메일을 등록한다.

```bash
# 전역 사용자 이름 설정
git config --global user.name "홍길동"

# 전역 이메일 설정 (GitHub 계정 이메일과 동일하게)
git config --global user.email "gildong@example.com"
```

### 프로젝트별 설정

특정 프로젝트에서 다른 이름/이메일을 쓰고 싶다면 `--global` 없이 실행한다.

```bash
cd my-project
git config user.name "회사닉네임"
git config user.email "work@company.com"
```

---

## 설정 확인

```bash
# 전체 설정 목록 보기
git config --list

# 특정 항목만 확인
git config user.name
git config user.email

# 설정 파일 직접 열기
git config --global --edit
```

---

## 기본 에디터 설정

```bash
# VS Code로 설정
git config --global core.editor "code --wait"

# Vim (기본값)
git config --global core.editor "vim"

# Nano
git config --global core.editor "nano"
```

---

## 기본 브랜치 이름 설정

```bash
# 새 저장소 생성 시 기본 브랜치를 main으로 설정
git config --global init.defaultBranch main
```

---

## 줄바꿈 설정 (CRLF / LF)

팀원 간 OS가 다를 때 줄바꿈 문자 충돌을 방지한다.

```bash
# macOS / Linux
git config --global core.autocrlf input

# Windows
git config --global core.autocrlf true
```

---

## SSH 키 설정

HTTPS 대신 SSH로 GitHub에 접속하면 매번 비밀번호를 입력하지 않아도 된다.

```bash
# 1. SSH 키 생성
ssh-keygen -t ed25519 -C "gildong@example.com"

# 2. SSH 에이전트에 키 등록
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. 공개키 복사 (GitHub Settings > SSH Keys에 등록)
# macOS
pbcopy < ~/.ssh/id_ed25519.pub
# Linux
cat ~/.ssh/id_ed25519.pub

# 4. 연결 테스트
ssh -T git@github.com
```

---

## GPG 서명 설정 (선택)

커밋에 서명을 추가하여 본인이 작성한 커밋임을 증명할 수 있다.

```bash
# GPG 키 생성
gpg --full-generate-key

# 키 ID 확인
gpg --list-secret-keys --keyid-format=long

# Git에 서명 키 등록
git config --global user.signingkey <KEY-ID>

# 모든 커밋에 자동 서명
git config --global commit.gpgsign true
```

---

## 유용한 별칭 (Alias) 설정

자주 쓰는 명령어를 짧게 줄일 수 있다.

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.unstage "restore --staged"
git config --global alias.last "log -1 HEAD"
git config --global alias.amend "commit --amend --no-edit"
```

사용 예시:

```bash
git st          # git status
git co main     # git checkout main
git lg          # 그래프 형태 로그
git amend       # 마지막 커밋 수정 (메시지 유지)
```

---

## 자격 증명 캐시

HTTPS 사용 시 매번 비밀번호 입력을 피하려면:

```bash
# 메모리에 15분간 캐시 (기본값)
git config --global credential.helper cache

# 캐시 시간 변경 (예: 1시간 = 3600초)
git config --global credential.helper 'cache --timeout=3600'

# macOS Keychain 사용
git config --global credential.helper osxkeychain

# Windows Credential Manager
git config --global credential.helper manager
```

---

## 설정 우선순위

Git 설정은 3단계로 적용되며, 좁은 범위가 넓은 범위를 덮어쓴다.

| 우선순위 | 범위 | 파일 위치 | 플래그 |
|---------|------|-----------|--------|
| 1 (높음) | 프로젝트 | `.git/config` | `--local` |
| 2 | 사용자 | `~/.gitconfig` | `--global` |
| 3 (낮음) | 시스템 | `/etc/gitconfig` | `--system` |

---

[다음: 저장소 관리 →](./02-repository.md)
