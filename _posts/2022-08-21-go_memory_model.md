---
layout: page
title: golang의 memory model
category : go
---

godev의 번역 글이다. 

고루틴의 메모리 모델은 한 고루틴이 읽는 변수가 다른 고루틴의 쓰기에서 생성되는 값을 관찰하는것에 대한 보장하는것의 조건을 설정한다.

여러고루틴이 동시에 엑세스해서 데이터를 수정하는 프로그램은 엑세스에대해서 직렬화가 필요하다.
직렬화를 위해서는 1. 채널을 사용, 2. 다른 동기화 요소(synchronization primitives)를 사용해야한다. (ex. sync패키지)


# 메모리 모델

메모리모델은 프로그램 고루틴으로 이루어진 프로그램 실행에 대한 요구사항을 기술하는데 memory operation으로 구성된다.

memory operation은 4가지 사항으로 모델링된다.

* 종류 : 이 작업이 일반적인 데이터 읽기, 쓰기 인지 아니면 atomic data access 나 mutex, channel operation같은 synchronizing operation 인지.
* 프로그램상의 위치
* 엑세스 중인 메모리 위치
* 그 작업에 의해 읽거나 써진 값.

어떤 memory operation은 read-like이다. (read, atomic read, mutex lock, channel receive)
다른 memory operation은 write-like이다. (write, atomic write, mutex unlock, channel send, channel close)

goroutine excution은 싱글 고루틴에서 실행되는 여러 memory operation으로 구성된다.

---
작성중

참조: https://go.dev/ref/mem