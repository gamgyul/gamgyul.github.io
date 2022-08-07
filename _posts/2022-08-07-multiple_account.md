---
layout: page
title: 한 기기에서 깃헙 계정 여러개 사용하기
category : 개발환경
---

노트북에서 깃헙 계정 여러개를 사용을 하게 되서 계정 설정하는법에 대해서 찾아본것을 기록한다.

## ssh 키 생성
깃헙의 연결인증은 ssh를 이용할것인데 mac에서 `ssh-keygen`을 사용해서 key를 생성한다.

```
ssh-keygen -t rsa
```

위의 명령어로 rsa를 이용한 ssh key가 만들어진다. 이때 파일 이름 설정을 하게되는데 두가지를 잘 나눠서 만들어주자.

## github에 키 등록

`깃헙 -> settings -> ssh and GPG keyx -> ssh key`에서 키를 등록하자

## ssh config 설정

```
  #user1 account
  Host github.com-user1
    HostName github.com
    User git
    IdentityFile ~/.ssh/github-user1
    IdentitiesOnly yes

  #user2 account
  Host github.com-user2
    HostName github.com
    User git
    IdentityFile ~/.ssh/github-user2
    IdentitiesOnly yes
```
이후에 위에사용한 호스트로 클론을 하면된다.

## git config 설정

```
git config --local user.name foo
git config --local user.email foo@example.com
```
사용할 repo의 config에 유저이름과 이메일을 설정한다.

## github repo의 url설정
```
git@github.com-user1:user1/your-repo-name.git your-repo-name_user1
```
위와 같은 형태로 아까 설정한 ssh config의 호스트를 사용해서 url을 이용한다.

참조 : 
https://gist.github.com/Jonalogy/54091c98946cfe4f8cdab2bea79430f9
