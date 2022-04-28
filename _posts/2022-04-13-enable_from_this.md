---
layout: page
title: c++ enable_shared_from_this
category : C++
---

enable_shared_from_this를 보기전에 왜 이 키워드가 필요한지 아래 예제를 보자.

```cpp
class BadOne {
public:
	BadOne() {
		cout <<"BAD one constructor" <<endl;
	}
	~BadOne() {
		cout <<"BAD one destructor" <<endl;
	}
};

int main() {

	BadOne* foo = new BadOne();
	shared_ptr<BadOne> ptr1(foo);
	//error
	shared_ptr<BadOne> ptr2(foo);

	return 0;
}
```

위코드를 돌려보면 아래와 같은 에러가 발생한다.

```  cmd
/enable_test.out 
BAD one constructor
BAD one destructor
BAD one destructor
free(): double free detected in tcache 2
Aborted
```

위와 같이 같은 raw pointer로 생성된 쉐어드 포인터가 중복할때 소멸자가 중복 호출이 되어서 문제가 발생한다.

또 다음과 같은 케이스를 사용해서도 에러가 난다. 

```cpp 
void Print() {
		if(GetSharedPtr().use_count() == 3)
			cout <<  "Doing Stuff" << GetSharedPtr().use_count() << endl;
		else
			cout <<  "Doing Bad" << GetSharedPtr().use_count() << endl;
	}

std::shared_ptr<BadOne> GetSharedPtr() { return std::shared_ptr<BadOne>(this); }
}

int main() {

	BadOne* foo = new BadOne();
	shared_ptr<BadOne> ptr1(foo);
	shared_ptr<BadOne> ptr2 = ptr1;
	
	ptr1->Print();
	return 0;
}
```

위 예제랑 마찬가지로 `GetSharedPtr()`시에 `this`로 생성하는 shared_ptr과 foo로 생성하는 shared_ptr이 컨트롤블록을 서로 모르기때문에 발생하는 문제이다. 

이러한 상황을 막기 위해서 enable_shared_from_this 키워드를 사용한다.
enable_shared_from_this 클래스의 내부를 확인해보면 

```cpp
mutable weak_ptr<_Tp>  _M_weak_this;
```

위와 같이 weak_ptr을 생성해서 나중에 이 weak_ptr을 사용한다.

enable_shared_frm_this 를 사용한 class는 아래와 같이 사용한다.

```cpp
class GoodOne : public std::enable_shared_from_this<GoodOne> {
public:
	GoodOne() {
		cout <<"Good one constructor" <<endl;
	}
	~GoodOne() {
		cout <<"Good one destructor" <<endl;
	}
	void Print() {
		if(GetSharedPtr().use_count() == 3)
			cout <<  "Doing Stuff" << GetSharedPtr().use_count() << endl;
		else
			cout <<  "Doing Bad" << GetSharedPtr().use_count() << endl;
	}

	std::shared_ptr<GoodOne> GetSharedPtr() { return shared_from_this(); }
};

int main() {

	GoodOne* foo = new GoodOne();
	shared_ptr<GoodOne> ptr1(foo);
	shared_ptr<GoodOne> ptr2 = ptr1;
	
	ptr1->Print();
	return 0;
}
```

결과는 아래와 같다.

```cmd
./enable_test.out 
Good one constructor
Doing Stuff3
Good one destructor
```

바뀐점은 클래스에서 enable_shared_from_this를 상속한것과 GetSharedPtr()을 만드는데 shared_from_this()라는 함수로 변경시켜준것이다.

주의 해야할 점으로 weak_ptr기반으로 생성된것이기 때문에 raw_pointer에 대해서 바로 만들 수 가없고 shared_ptr이 미리 존재해야한다.