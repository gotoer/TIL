# GitHub Push 실패? 권한 에러 해결법

### Push?

git push는 로컬 브랜치(local branch)를 원격 저장소(remote repository)로 푸시할 때 사용하는 기본 명령어입니다.

명령어는 아래와 같습니다.

`git push <remote> <branch>`

두 매개변수를 지정하지 않는다면 기본적으로 origin을 원격 저장소로, 현재 작업하고 있는 브랜치를 푸시할 브랜치로 지정합니다.

### Push 권한 에러

git clone으로 작업할 repository를 가져와서 코드 혹은 문서 작업을 완료하고 앞서 설명한 명령어를 통해 push를 한다고 해서 push가 가능한 것은 아닙니다.

코드 작성 후 commit하고 push를 해보면 다음과 같은 에러를 보게됩니다.

```
에러 예시:
git push origin main
remote: Permission to gotoer/TIL.git denied to kjh6759.
fatal: unable to access 'https://github.com/gotoer/TIL.git/': The requested URL returned error: 403
```

### 해결 방법

원격저장소에 접근할 권한을 갖고 있어야하며, 권한을 부여받기 위해 github에서 personal-access-token을 발급받아야 합니다.

1. github 페이지에서 우측 상단 이미지 클릭 후 하단 Settings를 클릭
2. 좌측 하단 Developer Settings 클릭
3. Personal Access Token 클릭 후 Fine-grained tokens 클릭
4. Generate New Token 버튼을 클릭 후 토큰 발급

### Access-Token 사용 방법

발급받은 토큰을 사용하는 방법은 여러가지가 있습니다.

1. 명령어에 직접 토큰을 입력
   : git clone 시 발급받은 personal-access-token을 명령어에 붙이는 방법입니다.

`git clone https://{PERSONAL_ACCESS_TOKEN}@github.com/gotoer/TIL.git`

이후 push는 인증절차 없이 진행할 수 있지만 토큰을 직접 입력하기 때문에 보안에 취약할 수 있습니다.

2. Git Credential Store 사용
   : git 자격 증명을 저장하여 매번 입력할 필요 없이 자동으로 인증하는 방법입니다.

`git config --global credential.helper store`

저장된 자격 증명이 평문으로 저장되므로 역시 안전하지는 않습니다.

`git config --global credential.helper cache`

cache 명령어를 통해 조금 더 안전하게 관리할 수는 있습니다.

3. SSH KEY 사용
   보안 측면에서 가장 우수하고 가장 권장하는 방식입니다.
   사용하기 위한 절차가 많다는 단점이 있습니다.

- SSH 키 생성
  : 사용하는 이메일을 입력하면 패스프레이즈를 입력할 수 있고 선택사항입니다. 패스프레이즈를 입력하면 보안이 더 강력해집니다.

```
ssh-keygen -t rsa -b 4096 -C "your@email.com"
```

- SSH 에이전트 실행 및 키 추가

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

- 공개 키 등록 (GitHub에 추가)
  : 아래 명령어를 입력하면 SSH key가 출력됩니다.
  출력된 키를 `GitHub 웹사이트 → Settings → SSH and GPG keys → New SSH key → 공개 키 붙여넣기 → 저장` 절차를 통해 저장합니다.

```
cat ~/.ssh/id_rsa.pub
```

- SSH 연결 테스트
  : 아래 명령어를 입력했을 때 `Hi <your-username>! You've successfully authenticated` 메시지가 나오면 성공입니다.

```
ssh -T git@github.com
```

뒤에 붙는 `but GitHub does not provide shell access` 메시지는 일반적으로 SSH 서버는 원격으로 접속해서 명령어를 실행할 수 있도록 쉘(Shell)을 제공하지만 GitHub의 SSH 서버는 Git 작업(Push/Pull 등)만 지원하고, 일반적인 SSH 명령어 실행은 허용하지 않음을 의미합니다.
결과적으로 인증절차는 성공했으니 위 문구는 넘어가도 됩니다.

### 결론

SSH KEY 사용하는 방법을 선택했고 원격 URL을 SSH 방식으로 변경하여 마무리 했습니다.
`git remote set-url origin git@github.com:gotoer/TIL.git`

`git remote -v` 명령어로 변경된 SSH URL까지 확인했습니다.

SSH KEY를 활용하는 것이 가장 안전하지만 위 방법 외에도 GIT_ASKPASS 환경 변수를 활용하는 등 다양한 방법이 존재합니다.
어찌됐든 대부분이 CLI를 통해 이뤄지기 때문에 linux에 대해 깊이있게 학습해야 하는 것을 느끼게 되었습니다.
