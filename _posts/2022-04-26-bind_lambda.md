---
layout: page
title: std::bind보다 lambda를 사용하자
category : C++
---

boost asio sample 코드에서 서버에서 this대신 `auto self(shared_from_this)`를 사용해서 Read를 하고 있어서 해당 부분이 궁금해서 찾다보니 [아래링크](https://stackoverflow.com/questions/34719233/why-capture-this-as-well-as-shared-pointer-to-this-in-lambdas)에서 std::bind와 lambda에 대해서 얘기가 나와서 추가적으로 더 찾아보기로 했다.(궁금했던건 위링크에 답변이 있었다.)

먼저 boost asio에 있던 std::bind (c++0x) 버전 과 lambda(c++11)의 예제를 보자.

```c++
  void start()
  {
    room_.join(shared_from_this());
    boost::asio::async_read(socket_,
        boost::asio::buffer(read_msg_.data(), chat_message::header_length),
        boost::bind(
          &chat_session::handle_read_header, shared_from_this(),
          boost::asio::placeholders::error));
  }

  void handle_read_header(const boost::system::error_code& error)
  {
    if (!error && read_msg_.decode_header())
    {
      boost::asio::async_read(socket_,
          boost::asio::buffer(read_msg_.body(), read_msg_.body_length()),
          boost::bind(&chat_session::handle_read_body, shared_from_this(),
            boost::asio::placeholders::error));
    }
    else
    {
      room_.leave(shared_from_this());
    }
  }
```
위 코드는 bind를 사용한 코드이다(boost::binid).  `async_read`에서 bind를 통해 함수와 인자들을 넣어주고 handler 함수에서 인자를 가지고 간다.

```c++
  void do_read_header()
  {
    auto self(shared_from_this());
    boost::asio::async_read(socket_,
        boost::asio::buffer(read_msg_.data(), chat_message::header_length),
        [this, self](boost::system::error_code ec, std::size_t /*length*/)
        {
          if (!ec && read_msg_.decode_header())
          {
            do_read_body();
          }
          else
          {
            room_.leave(shared_from_this());
          }
        });
  }
```

위코드는 c++ 11부터의 lambda를 사용한 함수인데 `async_read`에서 lambda를 마지막 인자로 넣어서 사용한다. 
위 두가지 경우만 비교해도 lambda를 사용한경우 훨씬 간결한것을 볼 수 있다.

bind 대신 lambda를 사용하자는 더 자세한 내용은 effective modern c++의  lambda 챕터에 설명이 되어있는데 알람을 생성하는 예제를 통해서 설명하고 있다.

## 1. 함수 호출 시점 문제
lambda를 사용한 케이스이다.

```c++    
    using Time = std::chrono::steady_clock::time_point;
    enum class  Sound {Beep, Siren, Whistle};
    using Duration = std::chrono::steady_clock::duration;
	
    auto	setSoundL = [](Sound s) {
		using namespace std::chrono;
		setAlarm(steady_clock::now() + hours(1), s, seconds(30));
	};
```

잘못된 케이스이지만 바인드의 경우는 아래와 같다,

```c++
auto setSoundB = std::bind(setAlarm, steady_clock::now() + hours(1),
							std::placeholders::_1, seconds(30));

```

위 케이스를 봤을때 인자로 `Sound`를 주기위해서 placeholder를 통해서 SetAlarm의 인자와 맵핑을 시켜야되고 이것만 하더라도 직관적이지 않다.

또한 lambda의 경우는 setAlarm이 호출될떄  `now() + hours(1)`이 결정이 되는데
bind버전의 경우 setAlarm이 호출될떄가 아닌 std::bind가 호출될때 결정된다.
이를 setAlarm이 호출될때로 변경하고 싶으면 아래와 같이 코드를 바꿔야한다.

```c++
auto setSoundB =
    std::bind(setAlarm,
    std::bind(std::plus<steady_clock::time_point>(),
    steady_clock::now(),
    hours(1)),
    _1,
    seconds(30));
```

## 2. 오버로딩 문제

인자가 바뀐 setAlarm함수가 오버로딩 됬다고 생각하자.

```c++
enum class Volume { Normal, Loud, LoudPlusPlus };
void setAlarm(Time t, Sound s, Duration d, Volume v);
```

이경우 lambda는 3가지 인자가 있는 버전을 선택하게 되지만 bind의 경우 compile error가 발생한다.

## 3. 직관성 문제

위에 조금 언급되긴하였지만 아래와 같은 예제로 다시 확인할 수 있다.

```c++
auto betweenL =
    [lowVal, highVal]
    (const auto& val) // C++14
    { return lowVal <= val && val <= highVal; };


auto betweenB = // C++11 version
    std::bind(std::logical_and<bool>(),
    std::bind(std::less_equal<int>(), lowVal, _1),
    std::bind(std::less_equal<int>(), _1, highVal));
```

## 4. call by value, reference 문제

```c++
enum class CompLevel { Low, Normal, High }; 

Widget compress(const Widget& w, 
    CompLevel lev); 
 Widget w;
using namespace std::placeholders;
auto compressRateB = std::bind(compress, w, _1);

auto compressRateL = 
    [w](CompLevel lev) // value; lev is
    { return compress(w, lev); }; 

    compressRateL(CompLevel::High); 

    compressRateB(CompLevel::High); 
 ```

 위의 `compressRateL` 람다를 보면 w가 캡쳐가 된것을 보면 값이 복사가 됬다는것을 알 수 있지만 `compressRateB` 바인드를 보면 ref 전달인지 value전달인지 구분을 하기가 힘들다.

 따라서 bind대신 lambda들 사용하기러 하자.
