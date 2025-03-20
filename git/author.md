# Git Author 변경

## 다른 계정으로 commit이 되어있다?

이전 포스팅에서 git repository를 만들고 push를 시도했지만 권한 문제가 발생해 해결하는 방법에 대해 다뤘습니다.
원격저장소에 push와 pull 작업은 문제없이 진행했는데 github을 들어가보니 다른 문제가 있었습니다.  

commit을 6번 정도 했는데 모두 회사계정으로 커밋이 되어있었고 그렇다보니 Contribution에 초록불이 들어오지 않은 것을 발견했습니다.  

<br>
<br>

## 문제의 원인

Git의 commit은 기본적으로 로컬 Git 설정에 저장된 사용자 정보(user.name과 user.email)를 기반으로 작성됩니다.  
문제는 회사계정에 `--global` 옵션이 적용되어 있었다는 점입니다. 해당 옵션을 설정하면 `~/.gitconfig` 파일에 저장되고 모든 저장소에 적용됩니다.  
그렇기 때문에 repository는 개인 계정으로 생성했지만 commit 했을 때는 회사계정으로 commit이 진행된 것입니다.

```
git config --global user.name
git config --global user.email
```

위 명령어를 치면 현재 Global로 설정된 계정을 확인할 수 있습니다.  

<br>
<br>

## 해결 방법

현재 개인개정으로 만든 repository에 user를 다르게 설정하면 됩니다.  
아래 명령어를 통해 쉽게 author를 변경할 수 있습니다.  

```
git config user.name "gotoer"
git config user.email kimbill6759@gmail.com
```

<br>

Global로 적용된 author를 바꾸고 싶다면 `--global` 옵션을 붙이면 됩니다.

```
git config --global user.name "gotoer"
git config --global user.email kimbill6759@gmail.com
```

<br>
<br>

## 이미 기존에 다른 계정으로 commit한 것을 바꾸려면?

git rebase 명령어를 사용해 author를 바꿀 수 있습니다.  
`git rebase -i HEAD~{다른 계정으로 커밋한 수}` 명령어를 수행하고 i를 눌러 insert 모드로 전환한 다음 "pick"을 "e"로 변경해줍니다.  
모두 변경을 완료했다면 insert 모드를 종료하고 :wq를 입력해 저장합니다.

<br>

이제 가장 먼저 진행한 commit부터 `git commit --amend --author="gotoer <kimbill6759@gmail.com>" -m "입력했던 커밋메시지"` 명령어를 실행해 author를 변경해줍니다.  
명령어 실행 후 `git rebase --continue`를 입력해 다음 commit으로 넘어가 동일한 작업을 완료합니다. 모든 변경이 완료되면 `git push -f origin {repository}`를 실행해 원격저장소에 변경된 사항을 반영합니다.
