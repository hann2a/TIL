# TIL: Git clone 직후 "modified 파일 수십 개" 뜰 때 (WSL/Windows 줄바꿈 문제)

## 현상

`git clone`(또는 checkout) 직후에 내가 아무 것도 안 했는데 `git status`에서 아래처럼 대량 변경이 잡힘:

- `Changes not staged for commit:`
- `modified: README.md` 등 수십~수백 개 파일

이 상태에서 `git diff`를 보면 **내용이 바뀐 것처럼 보이지 않는데** 파일 전체가 변경된 것으로 표시되는 경우가 많다.

---

## 원인 요약

대부분 **줄바꿈(Line Ending)** 이 자동으로 변환되어 발생한다.

- Windows: `CRLF` (Carriage Return + Line Feed)
- Linux / WSL: `LF` (Line Feed)

Git은 줄바꿈도 파일 내용의 일부로 보기 때문에, 체크아웃 또는 저장 과정에서  
`LF ↔ CRLF`가 바뀌면 파일 전체가 수정된 것으로 인식된다.

---

## 용어 정리

- **Working Tree**: 실제로 파일을 수정하는 작업 디렉토리
- **HEAD**: 현재 브랜치의 마지막 커밋
- **Stage(Index)**: `git add`로 커밋 대기 상태에 올린 영역
- **Line Ending (EOL)**: 텍스트 파일 줄 끝 문자
  - `LF (\n)` : Linux / macOS
  - `CRLF (\r\n)` : Windows
- **.gitattributes**: 저장소 단위로 줄바꿈·바이너리 정책을 강제하는 파일
- **core.autocrlf / core.eol**: Git의 줄바꿈 처리 방식 설정

---

## 진단 방법

### 1. diff가 의미 없는 변경인지 확인

```bash
git diff README.md | head -n 40
```

### 2. 줄바꿈 차이를 무시한 diff

```bash
git diff --ignore-space-at-eol --ignore-cr-at-eol -- README.md
```

→ 출력이 없으면 줄바꿈 문제

### 3. 실제 줄바꿈 확인

```bash
file README.md
```

- `with CRLF line terminators` → Windows 줄바꿈

### 4. Git 설정 확인

```bash
git config --show-origin --list | egrep "core.autocrlf|core.eol|eol|attributes"
```

---

## 해결 방법 (WSL 기준)

### 1. 저장소 단위 줄바꿈 설정

```bash
git config core.autocrlf input
git config core.eol lf
```

### 2. 원격 브랜치 상태로 복구

```bash
git fetch origin
git reset --hard origin/develop
```

> ⚠️ 로컬 변경사항이 있다면 모두 삭제되므로 주의

### 3. 확인

```bash
git status
file README.md
```

---

## 재발 방지 팁

- WSL 홈 디렉토리(`~/`)에서 작업 권장
- VSCode 우측 하단 줄바꿈 표시를 `LF`로 유지
- 팀 프로젝트라면 `.gitattributes` 도입 고려

---

## 한 줄 요약

**클론 직후 modified 폭탄은 대부분 CRLF/LF 줄바꿈 문제이며,  
WSL에서는 `core.autocrlf=input`, `core.eol=lf` 설정 후  
`git reset --hard origin/<branch>`로 해결 가능**
