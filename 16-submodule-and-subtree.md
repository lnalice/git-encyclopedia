# 16. Submodule & Subtree

[← 목차로 돌아가기](./README.md) | [← 이전: Cherry-Pick 심화](./15-cherry-pick.md)

---

## 외부 저장소를 프로젝트에 포함하는 두 가지 방법

| 항목 | Submodule | Subtree |
|------|-----------|---------|
| 구조 | 별도 저장소를 참조 링크로 포함 | 외부 코드를 직접 프로젝트 트리에 복사 |
| 저장소 | 부모/자식 저장소가 독립적 | 하나의 저장소에 통합 |
| clone | `--recurse-submodules` 필요 | 추가 작업 없이 바로 사용 |
| 커밋 이력 | 분리됨 (각자 별개 이력) | 통합됨 (하나의 이력) |
| 학습 곡선 | 높음 (함정이 많음) | 중간 |
| 적합한 경우 | 외부 라이브러리, 독립 배포가 필요한 모듈 | 공유 코드 통합, 모노레포 분리/통합 |

---

# Part 1: Submodule

## 개념

부모 저장소가 자식 저장소의 **특정 커밋 해시를 참조**하는 구조이다. 자식 저장소는 독립적인 Git 저장소로 유지된다.

```
my-project/               ← 부모 저장소
├── src/
├── libs/
│   └── shared-lib/       ← 서브모듈 (독립 저장소)
├── .gitmodules            ← 서브모듈 설정 파일
└── .git/
    └── modules/
        └── libs/shared-lib/  ← 서브모듈의 .git 데이터
```

---

## 서브모듈 추가

```bash
# 서브모듈 추가
git submodule add <repository-url> <path>
git submodule add https://github.com/team/shared-lib.git libs/shared-lib

# 특정 브랜치 추적 설정
git submodule add -b main https://github.com/team/shared-lib.git libs/shared-lib

# 추가 후 생성되는 파일
# .gitmodules — 서브모듈 설정 (경로, URL, 브랜치)
# libs/shared-lib — 서브모듈 디렉토리 (특정 커밋에 고정)
```

`.gitmodules` 내용:

```ini
[submodule "libs/shared-lib"]
    path = libs/shared-lib
    url = https://github.com/team/shared-lib.git
    branch = main
```

```bash
# 서브모듈 추가 커밋
git add .gitmodules libs/shared-lib
git commit -m "chore: shared-lib 서브모듈 추가"
```

---

## 서브모듈 포함 프로젝트 clone

```bash
# 방법 1: clone 시 서브모듈도 함께
git clone --recurse-submodules https://github.com/team/my-project.git

# 방법 2: 이미 clone한 경우 서브모듈 초기화
git submodule init
git submodule update

# 방법 2 한 줄로
git submodule update --init

# 중첩 서브모듈까지 모두 초기화
git submodule update --init --recursive
```

> **실수 방지**: clone 후 빌드 에러가 나면 서브모듈 초기화를 빼먹은 경우가 대부분이다.

---

## 서브모듈 업데이트

### 부모 저장소에서 서브모듈 최신화

```bash
# 모든 서브모듈을 원격의 최신 커밋으로 업데이트
git submodule update --remote

# 특정 서브모듈만
git submodule update --remote libs/shared-lib

# 업데이트 후 부모 저장소에 기록
git add libs/shared-lib
git commit -m "chore: shared-lib 최신 버전으로 업데이트"
```

### 서브모듈 안에서 직접 작업

```bash
# 서브모듈 디렉토리로 이동
cd libs/shared-lib

# 서브모듈은 기본적으로 Detached HEAD 상태
# 먼저 브랜치 체크아웃
git checkout main
git pull origin main

# 수정 & 커밋 (서브모듈 저장소에 커밋됨)
git add .
git commit -m "feat: 새 유틸 함수 추가"
git push origin main

# 부모 저장소로 돌아가서 참조 업데이트
cd ../..
git add libs/shared-lib
git commit -m "chore: shared-lib 커밋 참조 업데이트"
```

### 팀원이 서브모듈을 업데이트한 경우

```bash
# pull 후 서브모듈도 업데이트
git pull
git submodule update --init --recursive

# 또는 pull과 동시에
git pull --recurse-submodules
```

---

## 서브모듈 브랜치 전환

```bash
# 모든 서브모듈에서 특정 작업 실행
git submodule foreach 'git checkout main && git pull origin main'

# 특정 서브모듈의 특정 커밋으로 고정
cd libs/shared-lib
git checkout v2.0.0
cd ../..
git add libs/shared-lib
git commit -m "chore: shared-lib v2.0.0으로 고정"
```

---

## 서브모듈 삭제

서브모듈 삭제는 여러 단계를 거쳐야 한다.

```bash
# 1. 서브모듈 등록 해제
git submodule deinit -f libs/shared-lib

# 2. .git/modules에서 제거
rm -rf .git/modules/libs/shared-lib

# 3. 워킹 트리에서 제거
git rm -f libs/shared-lib

# 4. 커밋
git commit -m "chore: shared-lib 서브모듈 제거"
```

---

## 서브모듈 URL 변경

저장소가 이전되었거나 HTTPS → SSH로 바꿀 때.

```bash
# .gitmodules 파일에서 URL 수정
git config -f .gitmodules submodule.libs/shared-lib.url git@github.com:team/shared-lib.git

# 동기화
git submodule sync

# 재초기화
git submodule update --init --recursive

# 커밋
git add .gitmodules
git commit -m "chore: 서브모듈 URL 변경"
```

---

## 서브모듈 상태 확인

```bash
# 서브모듈 상태
git submodule status
#  abc1234 libs/shared-lib (v2.0.0)
# 접두사 의미:
#  (공백): 정상 — 부모가 기록한 커밋과 일치
#  +     : 서브모듈 HEAD가 부모 기록과 다름 (업데이트 필요)
#  -     : 서브모듈 미초기화

# 서브모듈 요약 (변경된 커밋 목록)
git submodule summary
```

---

## 서브모듈 주의사항

1. **Detached HEAD**: 서브모듈은 기본적으로 Detached HEAD 상태이다. 작업 전에 반드시 브랜치를 체크아웃해야 한다.
2. **커밋 순서**: 서브모듈 내부를 먼저 커밋+push하고, 그 다음 부모 저장소에서 커밋해야 한다. 순서가 바뀌면 팀원이 존재하지 않는 커밋을 참조하게 된다.
3. **clone 시 누락**: `--recurse-submodules` 없이 clone하면 서브모듈 디렉토리가 비어 있다.
4. **브랜치 전환 시**: 서브모듈이 있는 프로젝트에서 브랜치를 전환하면 서브모듈 상태가 꼬일 수 있다. 전환 후 `git submodule update`를 실행한다.

---

# Part 2: Subtree

## 개념

외부 저장소의 코드를 **직접 프로젝트 트리에 복사**하여 하나의 저장소로 관리한다. `.gitmodules` 같은 별도 설정 파일이 필요 없다.

```
my-project/               ← 하나의 저장소
├── src/
├── libs/
│   └── shared-lib/       ← subtree (프로젝트의 일부)
└── .git/                  ← 단일 .git
```

---

## Subtree 추가

```bash
# 외부 저장소를 원격으로 추가
git remote add shared-lib https://github.com/team/shared-lib.git

# subtree로 가져오기
git subtree add --prefix=libs/shared-lib shared-lib main --squash

# --prefix: 코드를 넣을 경로
# shared-lib: 원격 이름
# main: 가져올 브랜치
# --squash: 외부 이력을 하나의 커밋으로 합침 (권장)
```

### --squash 옵션

```bash
# squash 없이 — 외부 저장소의 전체 커밋 이력이 포함됨
git subtree add --prefix=libs/shared-lib shared-lib main

# squash 사용 — 하나의 커밋으로 합쳐서 깔끔하게 (권장)
git subtree add --prefix=libs/shared-lib shared-lib main --squash
```

> **실무 권장**: `--squash`를 사용하면 프로젝트 이력이 깔끔하게 유지된다. 다만, squash를 시작했으면 이후 pull/push에서도 계속 `--squash`를 사용해야 한다.

---

## Subtree 업데이트 (pull)

외부 저장소의 최신 변경사항을 가져온다.

```bash
# 원격 최신화
git fetch shared-lib

# subtree pull
git subtree pull --prefix=libs/shared-lib shared-lib main --squash

# 충돌 발생 시 일반적인 merge 충돌 해결과 동일
```

---

## Subtree에 변경사항 역전파 (push)

프로젝트 내에서 subtree 코드를 수정한 후 원본 저장소로 push할 수 있다.

```bash
# subtree의 변경사항을 원본 저장소로 push
git subtree push --prefix=libs/shared-lib shared-lib main

# 내부적으로 해당 경로의 커밋들을 분리하여 원격에 push
```

### split — 커밋 분리

subtree 경로의 커밋들만 별도 브랜치로 분리할 수 있다.

```bash
# libs/shared-lib 관련 커밋들을 별도 브랜치로 추출
git subtree split --prefix=libs/shared-lib -b shared-lib-split

# 분리된 브랜치를 원격에 push
git push shared-lib shared-lib-split:main
```

> `subtree push`가 느리다면 `split` 후 `push`하는 것이 효율적이다. `split`의 결과를 캐시하여 이후 작업이 빨라진다.

---

## Subtree 일상 워크플로우

### 프로젝트에 공유 라이브러리 추가

```bash
# 1. 원격 등록 + subtree 추가
git remote add ui-components https://github.com/team/ui-components.git
git subtree add --prefix=libs/ui-components ui-components main --squash
git push origin main

# 2. 팀원은 그냥 pull하면 됨 (submodule처럼 추가 초기화 불필요!)
git pull origin main
```

### 주기적으로 최신 코드 받기

```bash
git subtree pull --prefix=libs/ui-components ui-components main --squash
```

### 로컬에서 수정한 내용을 원본에 반영

```bash
# 프로젝트 내에서 libs/ui-components 코드를 수정/커밋한 후
git subtree push --prefix=libs/ui-components ui-components main
```

---

## 원격 등록 없이 사용

`git remote add` 없이 URL을 직접 지정할 수도 있다. 다만, 매번 URL을 입력해야 하므로 원격 등록을 권장한다.

```bash
git subtree add --prefix=libs/shared-lib \
    https://github.com/team/shared-lib.git main --squash

git subtree pull --prefix=libs/shared-lib \
    https://github.com/team/shared-lib.git main --squash
```

---

# Part 3: Submodule vs Subtree 비교

## 상세 비교

| 항목 | Submodule | Subtree |
|------|-----------|---------|
| **clone 시** | `--recurse-submodules` 필요 | 추가 작업 없음 |
| **팀원 학습 비용** | 높음 | 낮음 |
| **외부 이력 분리** | 완전 분리 | 합쳐지거나 squash |
| **독립 배포** | 가능 (각각 push/tag) | 가능하나 split 필요 |
| **디스크 크기** | 작음 (참조만) | 큼 (코드 복사) |
| **오프라인 빌드** | 불가 (fetch 필요) | 가능 (코드 포함) |
| **CI/CD** | 서브모듈 초기화 단계 추가 | 추가 설정 없음 |
| **코드 수정** | 서브모듈 안에서 별도 커밋 | 프로젝트에서 직접 수정 |
| **설정 파일** | `.gitmodules` 필요 | 없음 |

## 어떤 것을 선택할까?

### Submodule이 적합한 경우

- 외부 라이브러리를 **읽기 전용**으로 가져올 때
- 자식 저장소가 **독립적으로 버전 관리/배포**되어야 할 때
- 여러 프로젝트에서 **동일한 버전의 라이브러리**를 참조해야 할 때
- 저장소 크기를 최소화해야 할 때

```
사용 예: 팀 공통 프로토콜 버퍼 정의, 외부 SDK, 공유 설정 파일
```

### Subtree가 적합한 경우

- 공유 코드를 프로젝트에서 **직접 수정**하는 경우가 많을 때
- **팀원의 Git 숙련도**가 다양할 때 (submodule보다 실수가 적음)
- **CI/CD 파이프라인**을 단순하게 유지하고 싶을 때
- 모노레포에서 일부를 **분리하거나 통합**할 때

```
사용 예: 공유 유틸리티 라이브러리, 마이크로서비스 간 공통 코드, 모노레포 ↔ 멀티레포 전환
```

---

# Part 4: 실무 시나리오

## 시나리오 1: 마이크로서비스 간 공통 코드 공유

### Submodule 방식

```bash
# 각 서비스에서 common 저장소를 서브모듈로 추가
cd user-service
git submodule add git@github.com:team/common.git libs/common

cd ../payment-service
git submodule add git@github.com:team/common.git libs/common

# common 업데이트 시 각 서비스에서:
cd libs/common
git pull origin main
cd ../..
git add libs/common
git commit -m "chore: common 라이브러리 업데이트"
```

### Subtree 방식

```bash
# 각 서비스에서 common 저장소를 subtree로 추가
cd user-service
git remote add common git@github.com:team/common.git
git subtree add --prefix=libs/common common main --squash

cd ../payment-service
git remote add common git@github.com:team/common.git
git subtree add --prefix=libs/common common main --squash

# common 업데이트 시:
git subtree pull --prefix=libs/common common main --squash
```

## 시나리오 2: 모노레포에서 패키지 분리

하나의 큰 저장소에서 일부를 독립 저장소로 분리할 때 subtree split을 사용한다.

```bash
# 모노레포에서 packages/auth를 별도 저장소로 분리
git subtree split --prefix=packages/auth -b auth-standalone

# 새 저장소에 push
git remote add auth-repo git@github.com:team/auth-lib.git
git push auth-repo auth-standalone:main
```

## 시나리오 3: 서브모듈 → 서브트리 전환

서브모듈 관리가 번거로워서 서브트리로 전환할 때.

```bash
# 1. 서브모듈의 현재 커밋 확인
cd libs/shared-lib
git log -1 --oneline
# abc1234 현재 커밋

# 2. 서브모듈 제거
cd ../..
git submodule deinit -f libs/shared-lib
rm -rf .git/modules/libs/shared-lib
git rm -f libs/shared-lib
git commit -m "chore: shared-lib 서브모듈 제거"

# 3. 같은 경로에 subtree로 추가
git remote add shared-lib https://github.com/team/shared-lib.git
git subtree add --prefix=libs/shared-lib shared-lib main --squash
git commit -m "chore: shared-lib을 subtree로 재추가"
```

---

## 자주 발생하는 문제와 해결

### Submodule: "서브모듈 디렉토리가 비어있어요"

```bash
# clone 후 서브모듈 초기화를 안 한 경우
git submodule update --init --recursive
```

### Submodule: "Detached HEAD에서 작업해버렸어요"

```bash
cd libs/shared-lib
# 작업 내용이 Detached HEAD에 커밋된 상태
git log -1       # 커밋 해시 확인: abc1234
git checkout main
git cherry-pick abc1234
git push origin main
```

### Subtree: "pull 시 merge 충돌이 반복돼요"

```bash
# --squash를 사용했다가 빼거나 그 반대인 경우 발생
# 해결: 처음 add할 때 선택한 방식(squash 유무)을 일관되게 유지

# 심각한 경우 subtree 재추가
git rm -rf libs/shared-lib
git commit -m "chore: subtree 재설정을 위해 제거"
git subtree add --prefix=libs/shared-lib shared-lib main --squash
```

### Subtree: "push가 너무 느려요"

```bash
# split으로 캐시 생성 후 push
git subtree split --prefix=libs/shared-lib -b subtree-cache
git push shared-lib subtree-cache:main
```

---

[← 목차로 돌아가기](./README.md)
