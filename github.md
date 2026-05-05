# GitHub 연동 가이드

이 문서는 *초보자가 GitHub 계정을 만들고*, *`gh`를 설치하고*, *웹 브라우저 인증이 아닌 자동화 가능한 방식(SSH / 토큰)*으로 GitHub를 연동하는 방법을 정리한 문서입니다.

Hermes Agent나 다른 자동화 봇이 GitHub 작업을 수행하려면, 보통 *한 번만 사람이 설정*해 두고 이후에는 *토큰 또는 SSH 키*로 자동 수행하게 만드는 방식이 가장 안정적입니다.

---

## 1. GitHub 계정 만들기

GitHub를 사용하려면 먼저 계정이 필요합니다.

### 가입 절차

1. 아래 주소로 이동합니다.
   - https://github.com/signup
2. 이메일, 비밀번호, 사용자 이름을 입력합니다.
3. 이메일 인증을 완료합니다.
4. 보안 확인(퍼즐/인증)이 나오면 안내에 따라 진행합니다.
5. 가입이 끝나면 GitHub 프로필 페이지로 들어갑니다.

### 가입 후 확인할 것

- 사용자 이름이 원하는 값인지 확인
- 이메일 인증이 완료됐는지 확인
- 저장소를 만들거나 클론할 준비가 됐는지 확인

---

## 2. `gh` 설치 방법

`gh`는 GitHub CLI입니다. GitHub 작업을 명령줄에서 편하게 할 수 있게 해줍니다.

### Ubuntu / Debian 계열

```bash
sudo apt update
sudo apt install -y gh
```

### 설치 확인

```bash
gh --version
```

### 참고

- `gh`가 있으면 저장소 생성, PR, 이슈, 릴리즈, 인증 확인을 훨씬 쉽게 할 수 있습니다.
- 단, *브라우저 인증 없이 자동으로 돌리려면* 아래의 *토큰* 또는 *SSH* 방식이 더 적합합니다.

---

## 3. 인증 방식 선택하기

GitHub 인증은 크게 세 가지로 생각하면 됩니다.

| 방식 | 장점 | 추천 상황 |
|---|---|---|
| 브라우저 로그인 | 가장 쉬움 | 사람이 직접 로그인할 때 |
| 토큰(PAT) | 자동화에 좋음 | 봇 / 서버 / Termux / 비대화형 환경 |
| SSH 키 | git push/pull 자동화에 좋음 | 저장소 작업 위주 자동화 |

### 어떤 걸 쓰면 좋나?

- *저장소에 코드 올리기만 주로 한다* → SSH 키
- *GitHub API, PR, 이슈, 릴리즈까지 자동화하고 싶다* → 토큰(PAT) + `gh`
- *사람이 직접 로그인하는 방식은 피하고 싶다* → SSH 또는 토큰

---

## 4. 웹 인증이 아닌 자동화 방식 1: 토큰(PAT)

이 방식은 *봇이 자동으로 수행시키는 용도*에 가장 잘 맞습니다.

### 개념

- GitHub에서 개인용 액세스 토큰(PAT)을 발급
- 그 토큰을 `gh` 또는 환경변수로 저장
- 이후 GitHub 작업은 토큰으로 인증

### 토큰 만들기

GitHub → `Settings` → `Developer settings` → `Personal access tokens`

가능하면 *Fine-grained token*을 권장합니다.

필요 권한 예시:
- 저장소 코드 작업: `Contents`
- PR 작업: `Pull requests`
- 이슈 작업: `Issues`
- 액션/릴리즈까지 사용 시 필요한 권한 추가

### `gh`에 토큰으로 로그인

```bash
echo "YOUR_GITHUB_TOKEN" | gh auth login --with-token
gh auth setup-git
gh auth status
```

### 환경변수로 저장

자동화 봇이 쓰려면 토큰을 환경변수나 설정 파일에 저장해 둘 수 있습니다.

예:

```bash
export GITHUB_TOKEN="YOUR_GITHUB_TOKEN"
```

또는 `~/.hermes/.env` 같은 파일에 저장해 두고 로드할 수 있습니다.

```bash
GITHUB_TOKEN=YOUR_GITHUB_TOKEN
```

### 장점

- GitHub API를 직접 사용할 수 있음
- `gh`로 PR/이슈/릴리즈 자동화 가능
- 사람이 매번 로그인하지 않아도 됨

### 주의

- 토큰은 *채팅에 붙여넣지 않는 것*이 좋습니다.
- 권한은 *최소 권한*만 주는 것이 안전합니다.
- 토큰이 유출되면 즉시 재발급해야 합니다.

---

## 5. 웹 인증이 아닌 자동화 방식 2: SSH 키

SSH는 *git push/pull 같은 저장소 작업*을 자동화할 때 매우 좋습니다.

### SSH 키 생성

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/id_ed25519 -N ""
```

### 공개키 확인

```bash
cat ~/.ssh/id_ed25519.pub
```

### GitHub에 공개키 등록

GitHub → `Settings` → `SSH and GPG keys` → `New SSH key`

### 연결 테스트

```bash
ssh -T git@github.com
```

정상이라면 다음과 비슷한 메시지가 나옵니다.

```text
Hi username! You've successfully authenticated...
```

### Git 저장소를 SSH로 사용

```bash
git clone git@github.com:username/repo.git
```

기존 HTTPS 주소를 SSH로 자동 변환하려면:

```bash
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

### 장점

- 저장소 push/pull 자동화에 좋음
- 브라우저 인증이 필요 없음
- 장기적으로 안정적

### 한계

- GitHub API 작업(PR 생성, 이슈 생성 등)은 토큰 방식이 더 편함
- 저장소 작업 위주일 때 가장 적합

---

## 6. 봇이 자동으로 수행시키는 권장 구성

Hermes Agent 같은 봇이 자동으로 GitHub 작업을 하게 하려면, 보통 아래 구성이 가장 좋습니다.

### 추천 1: 토큰 + `gh`

이 방식은 *가장 범용적*입니다.

1. GitHub 토큰 발급
2. `gh auth login --with-token`
3. `gh auth setup-git`
4. 이후 저장소 / PR / 이슈 / 릴리즈 작업 자동화

### 추천 2: SSH + 토큰 병행

- *Git 저장소 작업* → SSH
- *GitHub API 작업* → 토큰

이 조합이 가장 실용적입니다.

### 자동화 흐름 예시

```text
사람이 1회 설정
→ SSH 키 등록 또는 PAT 발급
→ gh auth login --with-token
→ gh auth setup-git
→ 이후 봇이 자동 수행
```

---

## 7. 초보자를 위한 최소 순서

처음 하는 사용자라면 아래 순서가 제일 쉽습니다.

1. GitHub 계정 생성
2. `gh` 설치
3. `gh auth login` 또는 PAT 발급
4. 가능하면 `gh auth setup-git`
5. 자동화가 필요하면 SSH 키도 추가

---

## 8. 이 저장소의 현재 상태

이 문서는 아래 작업들을 정리한 기록입니다.

- `gh` 설치 및 인증
- GitHub 저장소 생성
- 로컬 저장소 연결
- 토큰 환경변수 저장
- Termux / Samsung S22 설치 기록

---

## 9. 참고

- GitHub 가입: https://github.com/signup
- GitHub CLI: https://cli.github.com/
- GitHub 토큰: https://github.com/settings/tokens
- SSH 키: https://github.com/settings/keys

---

## 10. 메모

이 문서는 *초보자용 GitHub 연동 문서*이면서, *봇이 자동으로 수행할 수 있는 인증 방식*을 함께 설명하는 용도로 작성되었습니다.
