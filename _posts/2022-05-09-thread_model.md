---
layout: page
title: 멀티쓰레드 모델
category : CS
---

최근에 go를 공부를 하는데 고루틴이 M:N multithread model을 사용해서 멀티코어를 활용을 잘하고 context switching 비용도 적다는 글을 봐서 운영체제 공부할때 리눅스나 윈도우는 1:1을 사용하는구나 하고 넘어갔던 기억이 있어서 왜 m:n 이 좋다고 하는데 리눅스는 1:1이지 하는 궁금증이 생기고 전에는 왜 넘어갔었는지 해서 좀 더 자세히 정리해보려고한다.
<!-- 왜 언어는 m:n인데 OS는 1:1이 될수있지 -->

## user thread, kernel thread

먼저 M,N에 해당하는 `user thread`와 `kernel thread`의 정의를 보자. 
어떤 문서에서는 `kernel thread`를 사용자가 생성한것이 아닌 OS의 background에서 도는 스레드이고 `user thread`는 사용자가 생성한 thread라고 표현을 하지만 multithreading model에서 말하는 `user thread`와 `kernel thread`는 layer 개념이여서 위 표현 보다는 다르게 생각해야 하는것 같다.

### user thread(M)

user thread는 사용자가 설정하는 동시성을 나누는 단위를 할한다. 즉 thread API를 호출하는 것을 말한다. 예를들어 c++에서 std::thread를 호출하던가 java에서 runnable의 start를 호출하는것과 같은 app단에서 호출하는 thread를 말한다.

### kernel thread(N)

kernel thread는 스케줄러에 스케줄링이 되는 스레드를 말한다. 즉 logical core에 할당되는 단위를 말한다.

## M:1, 1:1, M:N  multithread model

굉장히 헷갈렸던것이 인터넷 문서들을 보면 user thread의 장단점을 말할때 user thread는 context switching이 빠르지만 한쓰레드가 블락킹 되면 다른쓰레드도 블락킹이 된다는것이 였다. 내가 c++로 thread를 생성하면 io 대기중일때 다른 스레드도 같이 블락킹이 된다는 말인가?. 좀더 확인해보니 `user_thread`라는 말으 의미를 혼용해서 써서 헷갈렸던것이였다. M:N에서의 M에 대해서도 `user_thread의 갯수로 표현하기도 하지만 위 의문에서 말하는 user thread는 (N:1) user level threading을 말하는것이고 m:n의 m은 사용자 API자체만을 말한다. c++의 경우 linux에서는 pthread_create를 호출해서 Native POSIX Thread Library(NPTL)을 통해서 kernel thread를 생성을 하기떄문에 사용자 API에서 호출하나당 kernel thread가 한개 생성되기떄문에 1:1 모델이 되고 NPTL이 linux에서 제공하는 기본 라이브러리이기 때문에 책에서 리눅스는 1:1모델이라고 했던것이다. 이 1:1 kernel level threading 자체를 kernel thread라고 부르기도 한다.


## Goroutine은 어떻게 M:N인것인가

처음에 고루틴에 대해 들었을때 든 생각이 Linux가 1:1인데 고루틴은 어떻게 M:N이 될 수 있지를 생각이였었다. M:N에서 M을 나타내는 말이 user thread라고 했지만 위에 언급했던것처럼 사실 M은 사용자 layer에서의 동시성에 해당하는 주체(사용자가 API를 호출하는 것)이고 golang에서의 GMP중 고루틴이 사용자가 설정하는 동시성을 나누는 단위이기 때문이다. 이 고루틴이 커널의 스케쥴러에 1:1맵핑이 된것이 아니고 go runtime scheduler에 의해서 스케쥴링되어 n개의 kernel 쓰레드를 clone을 통해 생성하기 떄문에 전체적으로 보았을때 M:N 모델이 된다.


참고 자료:

what-is-the-difference-between-nptl-and-posix-threads
https://en.wikipedia.org/wiki/Thread_(computing)#M:N_.28hybrid_threading.29
https://sungjunyoung.github.io/posts/how-goroutine-works/