---
layout: page
title: golang의 waitgroup
category : go
---

waitgroup은 goroutine 집단이 종료될때까지 대기를 하는 그룹이다. 메인 고루틴이 `Add`함수를 콜해서 wait에 걸릴 고루틴들을 추가시켜주고 각각의 고루틴에서는 `Done`을 호출해서 고루틴의 마무리를 알린다. 이후 `wait`은 모든 고루틴이 마무리될때까지 기다리게 된다.

C/C++을 생각하면 thread join을 생각하면 될것 같다.

```go
package practice

import (
	"fmt"
	"sync"

	"github.com/valyala/fasthttp"
)

func GetFromUrl(url string, cl *fasthttp.Client, wg *sync.WaitGroup) {
	defer wg.Done()
	status, _, err := cl.Get(nil, url)
	if err != nil {
		fmt.Println(err.Error())
	} else {
		fmt.Println("status: ", status)
	}
}

func WaitGroupPractice() {
	var (
		urls = []string{
			"http://www.golang.org/",
			"http://www.google.com/",
			"http://www.example.com/",
		}
		wg sync.WaitGroup
	)
	testClient := &fasthttp.Client{}
	for _, url := range urls {

		wg.Add(1)
		go GetFromUrl(url, testClient, &wg)
	}
	wg.Wait()
}
```

go.dev의 예제를 수정한 예제이다. 예제를 보면 url루프를 돌면서 `GetFromUrl`이라는 함수를 고루틴으로 추가하고 있는데 함수를 호출하기전에 `waitgroup.Add`를 호출해서 해당 고루틴을 waitgroup에 등록하고 호출된 함수 내에서는 `defer`를 통해서 goroutine이 종료될때  `Done` 함수를 호출한다.


## Add()

```go
func (wg *WaitGroup) Add(delta int)

```
Add는 위에 언급한것처럼 wait에 걸릴 고루틴을 추가하는 함수 이고 안의 delta는 마이너스가 될 수 도 있다. 그렇지만 `Wait`함수가 호출되기전까지는 총 delta가 양수가 되어야지 wait이 걸리고 음수가 되면 패닉이 발생한다.