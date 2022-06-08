---
layout: page
title: golang의 defer, panic, recover
category : go
---
golang을 공부하면서 본 defer, panic, recover 관련 godev 블로그글을 번역및 정리 하려고한다. 


# defer

defer문은 함수를 list에 넣은후 그 리스트를 현재 실행하는 함수가 리턴될때 실행한다.
주로 자원 정리를 위해 사용을 한다.(c++ 의 소멸자와 비슷한 역할?)

사용예를 아래와 같이 확인하자. 
``` go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

이 코드는 만약 Create함수에서 호출이 실패를 하면 src파일의 close를 하지 않을 수 있는 상태에서 함수가 리턴될 수 있다. src.close의 위치를 옮겨주는것으로 해결 할 수 있지만 소스코드가 복잡해지면 문제를 찾기 쉽지 않아진다. `defer`를 사용하면 쉽게 처리할 수 있다.

``` go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

defer를 사용하면 close하는 문장을 open 뒤에 바로 써도되고 어떠한 return 조건을 타던 return시에 처리해줄 수 있게 해준다.

defer가 수행되는데는 몇가지 규칙이 있다.

1. defer가 정의된 문장에서의 인자값이 들어간다.

```go 
func a() {
    i := 0
    defer fmt.Println(i) // i가 뒤에 올라가지만 defer가 수행될때는 0을 출력한다. 
    i++
    return
}
```

2. defer가 수행될때는 LIFO로 가장 마지막에 넣은 defer부터 수행이된다.

3. deferred 함수는 현재 수행함수 return 값에 대한 접근 및 변경이 가능하다.

```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```

위 함수를 보면 defer함수가 c의 return 값인 i를 변경시켜서 값이 1이 아닌 2가 출력된다.
이방법은 error가 return 될때 값을 변ㄱ경해주는 좋은 방법이다.

# panic 

panic은 현재 흐름을 종료하고 패닉을 발생시킨다. 예를 들어 함수 F에서 패닉이 발생되면 함수F는 실행중이던 흐름을 멈추고 defered 함수를 수행한다. 그이후 함수F를 호출한함수에 리턴시킨다.호출한 함수입장에서는 F가 panic을 호출한것처럼 동작한다. 이후 프로세스는 현재 고루틴의 함수 스택을 올라가면서 리턴합니다. 패닉은 사용자가 직접 호출할 수 있으며 범위를 벗어난 배열 인덱스 접근과 같은 런타임 에러로도 발생할 수 있습니다.

# recover 

recover는 패닉상태의 고루틴에서 다시 제어를 가지는 함수입니다. recover는 defered함수에서만 작동합니다.. 정상상태의 recover호출은 nil을 반환하고 아무런 영향이 없습니다. 패닉상태의 고루틴에서는 recover가 패닉상태의 변수를 캡쳐하고 정상상태로 돌려놓습니다.

```go
package main

import "fmt"

func main() {
    f()
    fmt.Println("Returned normally from f.")
}

func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}
```

위 예제를 보면 함수 호출 순서가 다음과 같다.

<pre>Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
Recovered in f 4
Returned normally from f. </pre>

하나씩 살펴보면

1. f가 호출
2. g가 호출 (i <= 3이면 재귀적으로 g가 호출)
3. i가 4일때 panic이 발생
4. 패닉이 발생한후 함수스택을 따라서 올라가면서 defer함수를 수행
5. f의 defer함수를 수행할때 recover발생하고 이후 main함수에서 print함수가 수행된다.

또 추가적으로 실제 예로 json 패키지에서 발생하는 panic과 recover를 예시로 들수 있다. 

```go
// jsonError is an error wrapper type for internal use only.
// Panics with errors are wrapped in jsonError so that the top-level recover
// can distinguish intentional panics from this package.
type jsonError struct{ error }

func (e *encodeState) error(err error) {
	panic(jsonError{err})
}
```

json 패키지에서 에러가 발생했을때 위와 같이 jsonError라는 타입으로 wrapping해서 패닉을 발생시킨다.

```go
func (e *encodeState) marshal(v any, opts encOpts) (err error) {
	defer func() {
		if r := recover(); r != nil {
			if je, ok := r.(jsonError); ok {
				err = je.error
			} else {
				panic(r)
			}
		}
	}()
	e.reflectValue(reflect.ValueOf(v), opts)
	return nil
}
```

그러면 위의 실제 수행함수의 defer함수에서 recover를 수행한이후 panic타입이 jsonError이면 recover수행시 err값을 기록해 값을 전달하고 아니라면 다시 panic을 발생시킨다. (c++ 의 try catch throw에 대응되는데 조금더 편한방법으로 생각하면 될것으로 보임.)



참조 : https://go.dev/blog/defer-panic-and-recover