# Git & GitHub 실무 협업 명령어 백과사전

> 실무에서 팀과 협업할 때 필요한 Git/GitHub 명령어를 주제별로 정리한 레퍼런스입니다.  
> 필요한 상황에 맞는 문서를 찾아 빠르게 참고하세요.

---

## 목차

| # | 문서 | 설명 | 난이도 |
|---|------|------|--------|
| 01 | [초기 설정](./01-initial-setup.md) | Git 설치 후 가장 먼저 해야 할 설정들 | ★☆☆ |
| 02 | [저장소 관리](./02-repository.md) | 저장소 생성, 복제, 원격 연결 | ★☆☆ |
| 03 | [스테이징 & 커밋](./03-staging-and-commit.md) | 변경사항 추적, 스테이징, 커밋 | ★☆☆ |
| 04 | [브랜치 관리](./04-branch.md) | 브랜치 생성, 전환, 삭제, 전략 | ★★☆ |
| 05 | [원격 저장소](./05-remote.md) | push, pull, fetch 및 원격 관리 | ★★☆ |
| 06 | [병합 & 리베이스](./06-merge-and-rebase.md) | merge, rebase, cherry-pick | ★★☆ |
| 07 | [되돌리기](./07-undo-and-reset.md) | reset, revert, restore로 실수 복구 | ★★★ |
| 08 | [임시 저장 (Stash)](./08-stash.md) | 작업 중 임시 저장 & 복원 | ★★☆ |
| 09 | [로그 & 히스토리](./09-log-and-history.md) | 커밋 이력 조회, blame, diff | ★★☆ |
| 10 | [태그](./10-tag.md) | 릴리스 버전 태그 관리 | ★☆☆ |
| 11 | [충돌 해결](./11-conflict-resolution.md) | Merge conflict 해결 가이드 | ★★★ |
| 12 | [협업 워크플로우](./12-collaboration-workflow.md) | PR, 코드리뷰, Fork 기반 협업 | ★★☆ |
| 13 | [GitHub CLI](./13-github-cli.md) | `gh` 명령어로 GitHub 작업 자동화 | ★★☆ |
| 14 | [.gitignore & 파일 관리](./14-gitignore.md) | 추적 제외, LFS, .gitattributes | ★☆☆ |
| 15 | [Cherry-Pick 심화](./15-cherry-pick.md) | 커밋 선택 적용, 머지 커밋 pick, 실무 시나리오 | ★★★ |
| 16 | [Submodule & Subtree](./16-submodule-and-subtree.md) | 외부 저장소 통합, 비교, 전환 가이드 | ★★★ |

---

## 빠른 검색 가이드

### 상황별 바로가기

| 이런 상황이라면 | 참고 문서 |
|----------------|-----------|
| Git을 처음 설치했다 | [01-초기 설정](./01-initial-setup.md) |
| 새 프로젝트를 시작한다 | [02-저장소 관리](./02-repository.md) |
| 코드를 수정하고 저장하고 싶다 | [03-스테이징 & 커밋](./03-staging-and-commit.md) |
| 새 기능을 개발한다 | [04-브랜치 관리](./04-branch.md) |
| 팀원에게 코드를 공유한다 | [05-원격 저장소](./05-remote.md) |
| 브랜치를 합쳐야 한다 | [06-병합 & 리베이스](./06-merge-and-rebase.md) |
| 실수를 되돌리고 싶다 | [07-되돌리기](./07-undo-and-reset.md) |
| 급한 작업이 들어왔다 | [08-임시 저장](./08-stash.md) |
| 누가 이 코드를 썼는지 궁금하다 | [09-로그 & 히스토리](./09-log-and-history.md) |
| 릴리스 버전을 찍는다 | [10-태그](./10-tag.md) |
| 충돌이 발생했다 | [11-충돌 해결](./11-conflict-resolution.md) |
| PR을 올리고 리뷰를 받는다 | [12-협업 워크플로우](./12-collaboration-workflow.md) |
| 터미널에서 GitHub 작업을 한다 | [13-GitHub CLI](./13-github-cli.md) |
| 특정 파일을 Git에서 제외한다 | [14-.gitignore & 파일 관리](./14-gitignore.md) |
| 대용량 파일을 관리한다 (PSD, ZIP 등) | [14-.gitignore & 파일 관리](./14-gitignore.md) |
| 다른 브랜치의 특정 커밋만 가져오고 싶다 | [15-Cherry-Pick 심화](./15-cherry-pick.md) |
| 외부 저장소를 프로젝트에 포함하고 싶다 | [16-Submodule & Subtree](./16-submodule-and-subtree.md) |
| 버그가 어떤 커밋에서 생겼는지 찾고 싶다 | [09-로그 & 히스토리](./09-log-and-history.md) |
| 커밋 이력에서 민감 정보를 제거한다 | [07-되돌리기](./07-undo-and-reset.md) |
| 여러 브랜치를 동시에 작업하고 싶다 | [04-브랜치 관리](./04-branch.md) |
| 모노레포에서 일부만 체크아웃한다 | [02-저장소 관리](./02-repository.md) |
| 팀에 Git 훅을 적용하고 싶다 | [12-협업 워크플로우](./12-collaboration-workflow.md) |

---

## 표기법 안내

```
git command <필수-인자>           # 꺾쇠: 반드시 입력
git command [선택-인자]           # 대괄호: 생략 가능
git command (A | B)              # 파이프: 둘 중 하나 선택
git command --flag               # 더블 대시: 옵션 플래그
```

---

## 추천 학습 순서

**입문** → 01 → 02 → 03 → 05 → 14  
**중급** → 04 → 06 → 08 → 09 → 10 → 12  
**고급** → 07 → 11 → 13 → 15 → 16
