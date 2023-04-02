---
layout: page
title: golang의 empty structure에대한 잡설
category : go
---
golang의 empty struct
```golang
struct{}{}
```

처음본건 아래 코드처럼 c++의 set에 해당하는 map에 키가 있는지 체크하는 용도로 사용을 하는곳에서 봤다.

```golang
exist := make(map[string]structure{}{})
exist[key] = struct{}{}
if _, ok := exist[key]; ok {

}
```
이방법을 몰랐을때 `structure{}{}`대신 `boolean`을 사용했었는데 왜 사용을 하는지 찾아봤을때 empty struct가 메모리할당이 없기때문에 사용을 한다고 들었지만 성능향상이 미비하다고 해서 나같은 초보처럼 저게 어떤것인지 모르는 사람도 안찾아봐도 될 수 있게 원래 사용하던 `boolean`을 사용했었다. 그런데 항상 좋은 정보들을 많이 공유하시는 회사의 셀장님께서 흥미로운 글이라고 하시면서 [링크](https://www.reddit.com/r/golang/comments/gnge5m/stop_using_mapstringstruct_for_existence_checks/)를 주셨다. 그러다보니 왜이놈이 `structure{}{}`같이 생겼는지 궁금해졌고 이 [블로그](https://aidanbae.github.io/code/golang/emptystruct/)에서 좋은 참고 자료들을 주셔서 해당자료를 봤다. 
이런 자료들을 보면서 공부한점을 남겨본다.

## width

먼저 empty structure에 대해서 얘기하기전에 `width`를 먼저 이야기 하고 넘어갔었다.
`width`는 우리가 흔히 `size`라고 부르는 타입의 크기를 의미한다. golang에서 struct의 타입의 크기는 struct의 field들의 크기와 패딩의 합으로 정해진다. 이 padding의 크기를 결정하는것은 alignment인데 각요소들의 alignment중 큰값으로 결정이 난다.

`alignmemt`는 그타입이 직접사용되는 `general alignment` 그 값이 struct의 field로 사용되는 `field alignment`로 나누어지는데 go 1.20에서는 두값이 동일하다는것을 보장한다고 한다.

이값은 아래 코드처럼 `unsafe.Alignof(t)`로 컴파일타임에 알수 있고 런타임의 경우 `reflect.TypeOf(t).Align()`을 통해 알 수 있다. 아래 string과 complex은 size8의 합으로 구성되었기에 size가 16이여도 align은 8이다. (64bit 기준)

```golang
type Test struct {
	a string
	b int32
	c complex128
}

func main() {
	var a Test

	fmt.Println("size")
	fmt.Println(unsafe.Sizeof(a))
	fmt.Println(unsafe.Sizeof(a.a))
	fmt.Println(unsafe.Sizeof(a.b))
	fmt.Println(unsafe.Sizeof(a.c))

	fmt.Println("align")
	fmt.Println(unsafe.Alignof(a))
	fmt.Println(unsafe.Alignof(a.a))
	fmt.Println(unsafe.Alignof(a.b))
	fmt.Println(unsafe.Alignof(a.c))

	fmt.Println("reflect")
	fmt.Println(reflect.TypeOf(a).Align())
	fmt.Println(reflect.TypeOf(a).FieldAlign())
}
// output
// size
// 40 -> 16 + 8 + 16
// 16
// 4
// 16
// align
// 8
// 8
// 4
// 8
// reflect
// 8
// 8
```

## empty struct 
`width`는 위와같이 struct안의 타입들로 결정이나는데 `empty struct`란 `struct{}`로  표현되고 `width`가 0인 struct이다.

`empty struct`는 다른 변수들과 같이 마찬가지로 array나 slice로 생성이 될수 있는데 array일때도 width가 0이고 slice도 slice header크기만 존재한다.
크기도 0여서 주소도 같은 곳을 가르키고있다. 즉 `empty struct`들은 구분이 안된다.
그럼 이 `empty struct`를 어디에 사용을할까?


```golang 
func main() {
	var a struct{}
	var b int32
	var c struct{}
	var d struct{}

	fmt.Printf("%p %p %p %p", &a, &b, &c, &d)
}
// output : 0x559f08 0xc00001c030 0x559f08 0x559f08
```

## 사용처
다른언어에서 boolean으로 사용법한 곳에서 사용할 수 있다.
### 1. key가 존재하는지 확인

```golang
colorMap := make(map[string]struct{})
colorMap["red"] = struct{}{}
coloMap["green"] = struct{}{}

if _, ok := colorMap[x] {
	//
}
```

위의 코드처럼 어떤 키가 있는지 없는 지 확인하는 용도로 사용이 가능하다.

### 2. interface의 구현체로 사용

```golang
type Animal interface {
	Leg() int
	Eye() int
}

type dog struct{}
func(d *dog) Leg() int {
	return 4
}
func(d *dog) Eye() int {
	return 2
}
```

객체 지향적으로 생각했을때 변수가 따로 존재하지 않고 method의 집합으로만 이루어진 클래스를 생성할때 empty struct로 타입을 생성하고 메소드를 구현하는 방법을 사용할 수 있다.

### 3. 채널의 signal로 사용

채널을 사용할때 채널의 전달했다는 사실자체를 사용하는 경우가 있다.

```golang
func worker(mainCh chan int, quitCh chan struct{}) {
	for {
		select {
			case i:= <-mainCh:
				fmt.Println("value", i)
			case <-quitCh:
				chQuit <- struct{}{}
				return
		}
	}
}

func main() {
	mainCh := make(chan int)
	quitCh := make(chan struct{})
	go worker(mainCh, quitCh)
	mainCh <- 100
	quitCh <- struct{}
	
	// worker에서 작업이 끝나는것을 대기
	<-quitCh
}
```
위와 같은 코드에서 `quitCh`에서는 채널에 값이 들어왔다라는 사실만 중요한경우는 emptyStruct를 사용하는 채널을 이용할 수 있다.


# Reference
https://go101.org/article/memory-layout.html
https://go.dev/ref/spec#Size_and_alignment_guarantees
https://medium.com/@matryer/cool-golang-trick-nil-structs-can-just-be-a-collection-of-methods-741ae57ab262
