---
layout: page
title: golang이 c랑 syntax 선언이 다른 이유
category : go
---

 위 제목에 대한 내용을 go blog 문서에서 이유를 설명하고 있었다. go를 공부하면서 c랑 타입선언이 바뀌어 있는게 헷갈렸고 왜 그렇게 했을까 궁금했었는데 해당 문서를 번역및 정리를 하려고한다.

## C syntax

c는 타입을 왼쪽 이름을 오른쪽에 두고 사용한다.

``` c
int x;
int *p;
int a[3];

int main(int argc, char* argv[]);
```

그런데 타입이 복잡해지면 헷갈리기 시작한다고 한다 . 예를 들어 함수 포인터를 생각하면 

```c
int (*fp) (int a, int b) // 이정도는 알아볼수 있다.
// 그런데 함수포인터 의 인자로 함수가 들어간다면?..
int (*fp) (int (*ff) (int x, int y), int b)
```

이렇게 읽기가 어려워진다. c의 경우 declaration 하는 곳에서 함수의 인자 이름을 생략하기도 하는데 

```c
int (*fp)(int (*)(int, int), int)

// function pointer를 return을하면?

int (*(*fp)(int (*)(int, int), int))(int, int)
```

이렇게 보기가 어렵게 된다.

## Go syntax

```go
x int
p *int
a [3]int
```

Go의 Syntax는 이름이 왼쪽 타입이 오른쪽에 있는 식이고 왼쪽부터 오른쪽으로 읽으면 된다.
함수의 경우도 c의 main함수를 go식으로 바꾼것을보자.

```go
func main(argc int, argv []string) int 
```
함수도 왼쪽 부터 오른쪽으로 읽으면된다.

위쪽 C예제의 함수포인터를 사용하는 함수의 경우에는 

```go
func(func(int,int) int, int) int
```
가 되고 function pointer를 return 하는 경우는

```go
func(func(int,int) int, int) func(int, int) int
```

이렇게 넣는 인자와 return 값을 왼쪽부터 읽으면 되서 인식하기가 편하다.


실제로 생각해보니 함수를 인자로 넣을때 바로 타입을 집어넣지 않고 c++에서는 아래와 같이 using이나 typedef를 이용해서 나타낸 후에 사용을 했었다.
```c++
using SomeFunc = std::function<int (int argc, char*[] argv)>
```

언어를 만들때는 참 여러가지 고려를 많이 하는것같다.


참조 : https://go.dev/blog/declaration-syntax