---
layout: page
title: gitblog development
---

기존에 회사에서 위키를 사용을 하다가 이직을 하게 되면서 내가 개인적으로 저장을 할 공간이 필요하다고 느꼈고 그로 인해 깃허브로 블로그를 시작하면서 하는 작업들을 정리해본다.

## 깃블로그 설치 및 생성

1. 먼저 깃허브에 `${name}.github.io`에 해당하는 repository를 생성한다.  
다른 이름이여도 되지만 다른 설정없이 간단하게 사용할 수 있다.

2. jekyll을 통해서 블로그를 구성을 할텐데 jekyll은 ruby로 작성되어서 ruby를 먼저 설치를하여야 한다.   
그이후 아래와 같은 명령어로 설치를 할 수 있다.

``` cmd
$ gem install jekyll bundler 
```
    gem은 우분투의 apt, mac의 brew와 같이 패키지 관리 툴이고, bundler의 경우 프로젝트에 해당되는 패키지들의 의존성을 관리 해주는 패키지라고 한다.

3. 블로그의 모든것을 구성할 수 없기에 미리 존재하는 테마들을 사용할 수 있다.  
  아래 사이트에서 많은 도움을 받을 수 있다.
    
    https://jekyllrb-ko.github.io/resources/

4. 이후 clone 받은 directory에 가서 `bundle exec jeckyll serve` 명령어를 사용해서 
테스트를 해보자.

  