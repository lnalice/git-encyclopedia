# 09. 로그 & 히스토리

[← 목차로 돌아가기](./README.md) | [← 이전: 임시 저장](./08-stash.md)

---

## log — 커밋 이력 조회

```bash
# 기본 로그
git log

# 한 줄씩 요약
git log --oneline

# 최근 n개 커밋만
git log -5
git log --oneline -10

# 그래프 형태로 (브랜치 분기/병합 시각화)
git log --graph
git log --graph --oneline --all --decorate

# 변경된 파일 목록 포함
git log --stat

# 변경 내용(diff) 포함
git log -p
git log --patch

# 특정 파일의 이력
git log -- path/to/file.txt
git log --oneline -- src/auth/

# 특정 파일의 이력 + 변경 내용
git log -p -- path/to/file.txt

# 파일 이름 변경(rename) 이력까지 추적
git log --follow -- path/to/file.txt

# 머지 커밋의 첫 번째 부모만 따라가기 (main 브랜치 흐름 파악)
git log --first-parent --oneline
```

> **--first-parent**: merge가 많은 프로젝트에서 main 브랜치의 핵심 흐름만 볼 때 유용하다. PR merge 커밋만 순서대로 나열된다.

---

## 검색 & 필터링

```bash
# 커밋 메시지로 검색
git log --grep="로그인"
git log --grep="fix" --oneline

# 대소문자 무시
git log --grep="login" -i

# 코드 변경 내용으로 검색 (특정 문자열이 추가/삭제된 커밋)
git log -S "functionName"
git log -S "TODO" --oneline

# 정규식으로 코드 검색
git log -G "function\s+login"

# 작성자로 필터링
git log --author="홍길동"
git log --author="gildong@example.com"

# 날짜 범위
git log --since="2024-01-01"
git log --until="2024-06-30"
git log --since="2 weeks ago"
git log --since="3 days ago" --until="1 day ago"

# 여러 조건 조합
git log --author="홍길동" --since="2024-01-01" --grep="feat" --oneline
```

---

## 포맷 커스터마이징

```bash
# 커스텀 포맷
git log --pretty=format:"%h %an %ar %s"
# abc1234 홍길동 3 hours ago feat: 로그인 구현

# 자주 쓰는 포맷 옵션
# %H  전체 커밋 해시
# %h  축약 해시
# %an 작성자 이름
# %ae 작성자 이메일
# %ar 작성 상대 시간
# %ad 작성 날짜
# %s  커밋 메시지 제목
# %b  커밋 메시지 본문
# %d  브랜치/태그 정보

# 날짜 포맷 지정
git log --pretty=format:"%h %ad %s" --date=short
# abc1234 2024-03-15 feat: 로그인 구현

git log --pretty=format:"%h %ad %s" --date=format:"%Y-%m-%d %H:%M"
```

### 실무에서 유용한 로그 alias

```bash
# 예쁜 그래프 로그
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --all
```

---

## 브랜치 간 비교

```bash
# main에는 없고 feature에만 있는 커밋
git log main..feature/login --oneline

# feature에는 없고 main에만 있는 커밋
git log feature/login..main --oneline

# 양쪽 차이 모두 (어느 브랜치 소속인지 표시)
git log main...feature/login --oneline --left-right
# < abc1234 main에만 있는 커밋
# > def5678 feature에만 있는 커밋
```

---

## blame — 줄별 작성자 확인

각 줄을 누가, 언제, 어떤 커밋에서 수정했는지 확인한다.

```bash
# 파일 전체 blame
git blame file.txt

# 특정 줄 범위만
git blame -L 10,20 file.txt

# 공백 변경 무시
git blame -w file.txt

# 코드 이동/복사 감지
git blame -M file.txt     # 파일 내 이동 감지
git blame -C file.txt     # 다른 파일에서 복사 감지

# 특정 커밋 이전 blame
git blame <commit-hash> -- file.txt

# 이메일 표시
git blame -e file.txt
```

### 포매팅 커밋 무시 (--ignore-rev)

코드 포매터를 일괄 적용하면 blame 결과가 모두 포매팅 커밋으로 바뀌어 무의미해진다. 특정 커밋을 blame에서 제외할 수 있다.

```bash
# 특정 커밋 무시
git blame --ignore-rev <formatting-commit-hash> file.txt

# 여러 커밋을 파일로 관리 (팀 전체 적용)
# .git-blame-ignore-revs 파일 생성:
# abc1234  # 2024-03 prettier 일괄 적용
# def5678  # 2024-06 eslint --fix 일괄 적용

git blame --ignore-revs-file .git-blame-ignore-revs file.txt

# 기본 설정으로 등록 (매번 옵션 안 써도 됨)
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

> **실무 권장**: `.git-blame-ignore-revs` 파일을 저장소에 커밋하고 `blame.ignoreRevsFile`을 설정하면 팀 전체가 깔끔한 blame을 볼 수 있다. GitHub도 이 파일을 자동 인식한다.

---

## range-diff — 커밋 범위 비교

두 커밋 범위의 차이를 비교한다. force push된 PR에서 "이전 버전 대비 뭐가 바뀌었는지" 확인할 때 필수적이다.

```bash
# 기본 사용법: 이전 범위 vs 새 범위
git range-diff main..feature@{1} main..feature

# PR이 force push된 후 변경 확인
git fetch origin
git range-diff origin/main..origin/feature@{1} origin/main..origin/feature

# 3인자 형식: 공통 베이스, 이전 끝, 새 끝
git range-diff main feature@{1} feature

# 특정 reflog 항목으로 비교
git range-diff main feature@{2} feature@{0}
```

출력 예시:

```
1:  abc1234 = 1:  xyz9876 feat: 로그인 구현     (내용 동일, 해시만 변경)
2:  def5678 ! 2:  uvw5432 fix: 유효성 검사       (내용 변경됨)
3:  ghi9012 < -:  ------- style: 포맷 정리       (삭제됨)
-:  ------- > 3:  rst1111 test: 테스트 추가       (새로 추가됨)
```

> **리뷰어 필수 도구**: 리뷰 반영 후 force push된 PR을 다시 볼 때, 전체를 다시 읽지 않고 변경된 부분만 확인할 수 있다.

---

## show — 커밋 상세 정보

```bash
# 마지막 커밋 상세
git show

# 특정 커밋 상세
git show abc1234

# 특정 커밋의 특정 파일만
git show abc1234:path/to/file.txt

# 변경 통계만
git show --stat abc1234

# 태그 정보
git show v1.0.0
```

---

## shortlog — 작성자별 요약

```bash
# 작성자별 커밋 수
git shortlog -s -n

# 작성자별 커밋 목록
git shortlog

# 특정 범위
git shortlog v1.0..v2.0
```

---

## diff 심화

```bash
# 워드 단위 diff (코드 변경 하이라이트)
git diff --word-diff

# 변경된 함수/메서드 이름 표시
git diff --function-context

# 바이너리 파일 변경 확인
git diff --stat

# 줄 끝 공백 변경 무시
git diff -w
git diff --ignore-all-space

# 커밋 간 특정 파일 비교
git diff HEAD~3..HEAD -- file.txt
```

---

## reflog — HEAD 이동 이력

```bash
# HEAD 이동 기록 전체
git reflog

# 특정 브랜치의 reflog
git reflog show feature/login

# 날짜 포함
git reflog --date=iso

# 최근 n개
git reflog -10
```

---

## grep — 코드 내 텍스트 검색

Git이 추적하는 파일에서 텍스트를 검색한다. 일반 `grep`과 달리 Git이 관리하는 파일만 대상으로 하므로 `node_modules` 같은 디렉토리가 자동 제외된다.

```bash
# 현재 워킹 디렉토리에서 검색
git grep "TODO"

# 대소문자 무시
git grep -i "fixme"

# 줄 번호 포함
git grep -n "console.log"

# 파일명만 출력
git grep -l "deprecated"

# 특정 브랜치/커밋에서 검색
git grep "TODO" main
git grep "TODO" abc1234

# 정규식 검색
git grep -E "function\s+\w+Login"

# 특정 파일 타입에서만
git grep "import" -- "*.ts"

# 검색 결과에 함수명 표시
git grep -p "TODO" -- "*.py"
```

---

## bisect — 버그 도입 커밋 찾기

이진 탐색으로 버그가 처음 도입된 커밋을 자동으로 찾는다. 수백 개의 커밋 중에서도 빠르게 원인을 좁힐 수 있다.

```bash
# bisect 시작
git bisect start

# 현재(버그 있음)를 bad로 표시
git bisect bad

# 정상이었던 커밋을 good으로 표시
git bisect good abc1234

# Git이 중간 커밋을 체크아웃함 → 테스트 후 판정
git bisect good          # 이 커밋은 정상
git bisect bad           # 이 커밋에서 버그 발생

# 반복하면 원인 커밋을 찾음
# abc1234 is the first bad commit

# bisect 종료 (원래 브랜치로 복귀)
git bisect reset
```

### 자동 bisect (스크립트 활용)

```bash
# 테스트 스크립트로 자동 판정
git bisect start HEAD abc1234
git bisect run npm test

# 커스텀 스크립트
git bisect run ./test-bug.sh

# 성능 저하 원인 찾기
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
git bisect run ./benchmark.sh
# benchmark.sh: 성능 기준 미달 시 exit 1, 통과 시 exit 0
```

---

## notes — 커밋에 메모 추가

커밋 자체를 변경하지 않고 부가 정보를 남긴다. 코드 리뷰 결과, 테스트 상태 등을 기록할 때 유용하다.

```bash
# 현재 HEAD에 노트 추가
git notes add -m "성능 테스트 통과"

# 특정 커밋에 노트 추가
git notes add -m "리뷰 완료 - 승인" abc1234

# 노트 보기
git log --show-notes

# 노트 수정
git notes edit abc1234

# 노트 삭제
git notes remove abc1234

# 원격에 노트 push / fetch
git push origin refs/notes/*
git fetch origin refs/notes/*:refs/notes/*
```

---

## 실무 팁

### 특정 기능이 언제 도입됐는지 추적

```bash
# 코드에 특정 문자열이 추가된 커밋 찾기
git log -S "newFeatureFunction" --oneline

# 해당 커밋의 PR 번호 확인
git log -S "newFeatureFunction" --oneline | head -1
git show <hash>    # 커밋 메시지에서 PR 번호 확인
```

### 릴리스 간 변경 사항 확인

```bash
# 두 태그 사이의 커밋 목록
git log v1.0.0..v2.0.0 --oneline

# 변경 파일 목록
git diff v1.0.0..v2.0.0 --stat

# 작성자별 기여도
git shortlog v1.0.0..v2.0.0 -s -n
```

---

[다음: 태그 →](./10-tag.md)
