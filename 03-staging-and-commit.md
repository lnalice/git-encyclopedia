# 03. 스테이징 & 커밋

[← 목차로 돌아가기](./README.md) | [← 이전: 저장소 관리](./02-repository.md)

---

## 상태 확인

```bash
# 작업 디렉토리 상태 확인
git status

# 간결한 출력
git status -s
# M  modified-file.txt     (스테이징됨)
#  M unstaged-file.txt     (수정됨, 스테이징 안 됨)
# ?? new-file.txt          (추적되지 않는 파일)
# A  added-file.txt        (새로 추가됨)
# D  deleted-file.txt      (삭제됨)

# 브랜치 정보 포함
git status -sb
```

---

## 변경사항 비교 (diff)

```bash
# 워킹 디렉토리 vs 스테이징 영역
git diff

# 스테이징 영역 vs 마지막 커밋
git diff --staged
git diff --cached          # --staged와 동일

# 특정 파일만 비교
git diff -- path/to/file.txt

# 두 커밋 간 비교
git diff <commit1> <commit2>

# 두 브랜치 간 비교
git diff main..feature

# 변경된 파일 목록만 보기
git diff --name-only

# 변경 통계 (삽입/삭제 줄 수)
git diff --stat

# 커밋 전 공백/탭 오류 검사
git diff --check
git diff --cached --check    # 스테이징된 변경에 대해
```

> **실무 팁**: `git diff --check`를 pre-commit 훅에 넣으면 불필요한 공백 변경이 커밋되는 것을 방지할 수 있다.

---

## 스테이징 (add)

```bash
# 특정 파일 스테이징
git add file.txt

# 여러 파일 스테이징
git add file1.txt file2.txt

# 특정 디렉토리의 모든 변경사항
git add src/

# 모든 변경사항 스테이징 (새 파일 + 수정 + 삭제)
git add .
git add -A
git add --all

# 수정 및 삭제된 파일만 (새 파일 제외)
git add -u

# 대화형 스테이징 (파일 단위 선택)
git add -i

# 부분 스테이징 (같은 파일 내 일부분만)
git add -p file.txt
# y: 이 hunk 스테이징
# n: 건너뛰기
# s: 더 작게 분할
# q: 종료
```

### 부분 스테이징 활용

하나의 파일에 여러 변경이 섞여 있을 때, 논리적 단위로 분리해서 커밋할 수 있다.

```bash
git add -p
# 각 hunk마다 y/n으로 선택
git commit -m "feat: A 기능 추가"

git add -p
# 나머지 hunk 선택
git commit -m "fix: B 버그 수정"
```

---

## 스테이징 취소

```bash
# 특정 파일 언스테이징 (파일 내용은 유지)
git restore --staged file.txt

# 모든 파일 언스테이징
git restore --staged .

# 레거시 방식 (동일 동작)
git reset HEAD file.txt
```

---

## 커밋 (commit)

```bash
# 메시지와 함께 커밋
git commit -m "커밋 메시지"

# 스테이징 + 커밋 동시에 (추적 중인 파일만)
git commit -am "스테이징과 커밋을 한 번에"

# 에디터에서 긴 메시지 작성
git commit

# 빈 커밋 (CI 트리거 등에 활용)
git commit --allow-empty -m "trigger CI build"
```

---

## 커밋 메시지 수정

```bash
# 마지막 커밋 메시지 수정 (아직 push 안 한 경우)
git commit --amend -m "수정된 메시지"

# 마지막 커밋에 파일 추가 (메시지 유지)
git add forgotten-file.txt
git commit --amend --no-edit

# 에디터로 메시지 수정
git commit --amend
```

> **주의**: `--amend`는 커밋 해시를 변경한다. 이미 push한 커밋에는 사용하지 않는다.

---

## 커밋 메시지 컨벤션

팀에서 통일된 커밋 메시지 형식을 사용하면 이력 관리가 수월해진다.

### Conventional Commits 형식

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 주요 타입

| 타입 | 설명 | 예시 |
|------|------|------|
| `feat` | 새 기능 | `feat(auth): 소셜 로그인 추가` |
| `fix` | 버그 수정 | `fix(cart): 수량 음수 입력 방지` |
| `docs` | 문서 변경 | `docs: API 문서 업데이트` |
| `style` | 코드 포맷 (동작 변경 없음) | `style: 세미콜론 누락 수정` |
| `refactor` | 리팩토링 | `refactor(user): 서비스 레이어 분리` |
| `test` | 테스트 추가/수정 | `test(auth): 로그인 유닛 테스트 추가` |
| `chore` | 빌드, 설정 변경 | `chore: ESLint 설정 업데이트` |
| `perf` | 성능 개선 | `perf(query): 인덱스 추가로 조회 최적화` |
| `ci` | CI/CD 설정 | `ci: GitHub Actions 워크플로우 추가` |

### 좋은 커밋 메시지 예시

```bash
# 제목은 50자 이내, 명령형으로
git commit -m "feat(payment): 카카오페이 결제 연동

- 카카오페이 SDK 연동
- 결제 요청/승인/취소 API 구현
- 결제 실패 시 재시도 로직 추가

Closes #234"
```

---

## 파일 삭제 & 이름 변경

```bash
# Git에서 파일 삭제 (디스크에서도 삭제)
git rm file.txt

# Git 추적에서만 제거 (디스크에는 유지)
git rm --cached file.txt

# 디렉토리 삭제
git rm -r directory/

# 파일 이름 변경 / 이동
git mv old-name.txt new-name.txt
git mv file.txt src/file.txt
```

---

## 워킹 디렉토리 변경 취소

```bash
# 특정 파일 변경 취소 (마지막 커밋 상태로 복원)
git restore file.txt

# 모든 파일 변경 취소
git restore .

# 특정 커밋 시점으로 복원
git restore --source=<commit-hash> file.txt

# 레거시 방식
git checkout -- file.txt
```

> **주의**: `git restore`는 되돌릴 수 없다. 저장하지 않은 변경사항이 영구적으로 사라진다.

---

[다음: 브랜치 관리 →](./04-branch.md)
