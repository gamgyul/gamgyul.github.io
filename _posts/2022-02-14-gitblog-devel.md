---
layout: page
title: gitblog development
category : githubblog
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

4. 이후 clone 받은 directory에 가서 `bundle exec jekyll serve` 명령어를 사용해서 
테스트를 해보자.
http://localhost:4000에서 확인이 가능하다.

## 깃허브 블로그 오류 해결

1.  cannot locate Gemfile 오류
Gemfile이 없다고 해서 발생하는 오류  
  gitignore에 Gemfile을 등록 해서 발생.  
  생성하려면 다른 디렉토리에서 jeckyll new test를 통해서 생성  
  혹은 backup으로 만들어둔 .Gemfile.bak, .Gemfile.loc.bak 을 각각 Gemfile, Gemfile.lock으로 변경

2.  webrick 오류
  ```cmd
  C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
  ```
  Ruby 3.0.0 부터 webrick이 미포함이여서 발생하는 오류
  ```cmd
  $> bundle add webrick
  ```
  위 커맨드로 webrick을 추가하자.

## 페이지 추가
### 사이드바 페이즈 추가
<pre>
---
layout: page
title: gitblog development
---  
</pre>
페이지는 마크다운 형식으로 구성되며 위와 같이 layout과 title을 헤더로 파일가장 위에 작성한다. 사이드 바의 경우 sidebars 폴더 안에 작성해서 만들 수 있다. 
 
https://cookieshake.github.io/참고 하여 카테고리 적용한 sidebar적용

 ### 포스트 추가
<pre>
---
layout: post
title: What's Jekyll?
---
</pre>
위와 같이 헤더를 작성. home 화면에 포스트가 작성이 되는데 자세한건 더 알아봐야할듯.