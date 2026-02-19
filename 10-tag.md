# 10. 태그

[← 목차로 돌아가기](./README.md) | [← 이전: 로그 & 히스토리](./09-log-and-history.md)

---

## 태그란?

특정 커밋에 이름을 붙여 릴리스 버전을 표시한다. 브랜치와 달리 한 번 생성하면 이동하지 않는 고정 포인터이다.

---

## 태그 종류

| 종류 | 설명 | 특징 |
|------|------|------|
| Lightweight | 단순 포인터 | 이름만 존재 |
| Annotated | 작성자, 날짜, 메시지 포함 | GPG 서명 가능, 릴리스에 권장 |

---

## 태그 조회

```bash
# 태그 목록
git tag

# 패턴으로 검색
git tag -l "v1.*"
git tag --list "release-*"

# 태그 상세 정보
git show v1.0.0

# 태그와 커밋 함께 보기
git tag -n
git tag -n3              # 메시지 3줄까지 표시
```

---

## 태그 생성

### Lightweight 태그

```bash
git tag v1.0.0

# 특정 커밋에 태그
git tag v1.0.0 abc1234
```

### Annotated 태그 (실무 권장)

```bash
# 메시지와 함께 태그 생성
git tag -a v1.0.0 -m "첫 번째 정식 릴리스"

# 특정 커밋에 태그
git tag -a v1.0.0 -m "릴리스 v1.0.0" abc1234

# GPG 서명된 태그
git tag -s v1.0.0 -m "서명된 릴리스"
```

---

## 태그 푸시

태그는 `git push`로 자동 전송되지 않는다. 별도로 push해야 한다.

```bash
# 특정 태그 push
git push origin v1.0.0

# 모든 태그 push
git push origin --tags

# annotated 태그만 push
git push origin --follow-tags
```

---

## 태그 삭제

```bash
# 로컬 태그 삭제
git tag -d v1.0.0

# 원격 태그 삭제
git push origin --delete v1.0.0
git push origin :refs/tags/v1.0.0     # 축약형
```

---

## 태그 체크아웃

```bash
# 태그 시점으로 이동 (Detached HEAD)
git checkout v1.0.0

# 태그 기반으로 새 브랜치 생성
git checkout -b hotfix/v1.0.1 v1.0.0
git switch -c hotfix/v1.0.1 v1.0.0
```

---

## describe — 태그 기반 버전 서술

현재 커밋을 가장 가까운 태그 기준으로 서술한다. CI/CD에서 빌드 버전을 자동 생성할 때 핵심적으로 사용된다.

```bash
# 기본 사용법
git describe
# v1.2.0-14-gabc1234
# → v1.2.0 태그에서 14커밋 뒤, 현재 커밋 해시 abc1234

# annotated 태그만 사용 (기본값)
git describe --tags         # lightweight 태그도 포함

# 정확히 태그 위치에 있으면 태그명만 출력
git describe --exact-match  # 태그가 아니면 에러

# 항상 긴 형식 (태그 위에 있어도)
git describe --long
# v1.2.0-0-gabc1234

# dirty 표시 (커밋되지 않은 변경이 있으면)
git describe --dirty
# v1.2.0-14-gabc1234-dirty
```

### CI/CD 빌드 버전 예시

```bash
# 빌드 스크립트에서
VERSION=$(git describe --tags --always)
echo "Building version: $VERSION"
docker build -t myapp:$VERSION .

# --always: 태그가 없으면 커밋 해시 축약형 사용
git describe --always
# abc1234
```

---

## 서명 검증 (verify)

GPG 서명된 태그/커밋의 진위를 확인한다.

```bash
# 태그 서명 검증
git verify-tag v1.0.0

# 커밋 서명 검증
git verify-commit abc1234

# 로그에서 서명 상태 표시
git log --show-signature -5
```

---

## 시맨틱 버전(Semantic Versioning)

릴리스 태그에는 보통 `v메이저.마이너.패치` 형식을 사용한다.

```
v메이저.마이너.패치
v1.2.3

메이저(Major): 하위 호환이 깨지는 변경
마이너(Minor): 하위 호환을 유지하면서 기능 추가
패치(Patch):   버그 수정
```

### 예시

| 버전 | 의미 |
|------|------|
| `v0.1.0` | 초기 개발 |
| `v1.0.0` | 첫 정식 릴리스 |
| `v1.1.0` | 새 기능 추가 |
| `v1.1.1` | 버그 수정 |
| `v2.0.0` | 호환 깨지는 변경 |
| `v2.0.0-beta.1` | 프리릴리스 |
| `v2.0.0-rc.1` | 릴리스 후보 |

---

## 실무 팁

### 릴리스 워크플로우

```bash
# 1. release 브랜치에서 최종 테스트 완료 후
git switch main
git merge --no-ff release/v1.2.0

# 2. 태그 생성
git tag -a v1.2.0 -m "Release v1.2.0

- feat: 소셜 로그인 추가
- fix: 결제 오류 수정
- perf: 메인 페이지 로딩 최적화"

# 3. push
git push origin main --follow-tags
```

### 지난 릴리스 간 변경사항 확인

```bash
git log v1.1.0..v1.2.0 --oneline
git diff v1.1.0..v1.2.0 --stat
```

---

## archive — 소스 코드 압축 배포

릴리스 시 소스 코드를 압축 파일로 만들어 배포한다.

```bash
# tar.gz 아카이브
git archive --format=tar.gz --output=project.tar.gz HEAD

# zip 아카이브
git archive --format=zip --output=project.zip HEAD

# 특정 태그 기준
git archive --format=zip --output=release-v1.0.0.zip v1.0.0

# 특정 디렉토리만
git archive --format=zip --output=src.zip HEAD -- src/

# prefix 붙이기 (압축 해제 시 디렉토리 생성)
git archive --format=tar.gz --prefix=myapp-v1.0.0/ -o myapp-v1.0.0.tar.gz v1.0.0
```

---

## bundle — 저장소 오프라인 전송

네트워크 없이 저장소를 파일로 전송한다. 보안 환경이나 오프라인 상황에서 유용하다.

```bash
# 전체 저장소를 번들 파일로
git bundle create repo.bundle --all

# 특정 범위만 (증분 전송)
git bundle create update.bundle main ^origin/main

# 번들에서 복원
git clone repo.bundle my-project

# 번들에서 pull (증분 업데이트)
git pull update.bundle main

# 번들 유효성 검증
git bundle verify repo.bundle
```

---

[다음: 충돌 해결 →](./11-conflict-resolution.md)
