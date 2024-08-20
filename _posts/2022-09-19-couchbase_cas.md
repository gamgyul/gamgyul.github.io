---
layout: page
title: Couchbase의 CAS
category : DB
---
회사에서 CouchBase라는 DB를 쓰고 있는데 이 DB를 잘 몰라서 이 DB에 대해 일부 공부한것을 적어본다. 궁금증은 CAS에서 시작이 되었다.

## CouchBase에서 consistency를 지키는 방법

NoSQL은 rdb와 다르게 strong consistency를 지키지 않는다. rdb의 경우 동시에 다른 노드에서 데이터를 읽는 경우에 같은 데이터를 봐야만 한다. (즉 모든노드에 결과가 반영될때까지 commit이 되지 않는다.) 하지만 NoSQL의 경우 다른 노드에서 다른 데이터를 볼 수 가 있다. 

만약 그렇다면 A라는 데이터가 있고 1번노드와 2번노드에서 접근한 클라이언트가 각각 데이터를 B, C로 수정을 하려고 하면 먼저 수정한 데이터가 사라져버릴 수 있다. 이러한 문제를 CouchBase에서는 CAS라는 개념을 이용한다.
 
|순서| 1번노드| 2번노드 |
|-|-------|---------|
|1| A get |         |
|2|       | A get   |
|3| B Insert|       |
|4|       | C Insert|

위와 같이 B데이터가 날아감.


## CAS란

CAS는 멀티스레드 프로그래밍에서 나오는 lockfree, spinlock같은데서 나오는 `compare and swap`을 의미한다.transactionID와 같은 특정한 CAS수치가 다르면 Replace를 허용하지 않고 CAS mismatch 에러를 발생하고 같으면 값을 변경하고 CAS를 1올리는 방식이다. 아래는 couchbase에서 제공한 db에서 수행되는 CAS에 대한 pseudo코드이다.

```c
uint Replace(string docid, object newvalue, uint oldCas=0) {
    object existing = this.kvStore.get(docid);
    if (!existing) {
        throw DocumentDoesNotExist();
    } else if (oldCas != 0 && oldCas != existing.cas) {
        throw CasMismatch();
    }
    uint newCas = ++existing.cas;
    existing.value = newValue;
    return newCas;
}
```

위 예제 코드에서 CAS를 적용을 하면 아래와 같이 된다.



|순서| 1번노드| 2번노드 | CAS값 |
|-|-------|------|--------|
|1| A get | | CAS1 |
|2| | A get | CAS1|
|3| B Replace| | CAS1+1|
|4| | C Replace| 2번에서 가져온 Cas값이 지금이랑 달라 Replace 실패| 

위와 같이 에러를 발생시키게 된다.

## CAS Handling

이런식으로 `Replace`를 날렸을때 Cas mismatch에러가 발생하게 되는데 앱단에서는 mismatch가 날라왔을때 재수행 같은 로직을 구현해서 결국에는 모두 같은 값을 보는 eventual consistency는 만족 시킬수 있게 된다.

추가로 락도 제공한다고 하는데 couchbase에서 권장하는 방법은 아니라고 하고 자세히 보지는 않았다.


# DBCluster에서 Consistency

그런데 의문이 있다. CAS는 봤을때 한머신에서 동작하게 생긴 녀석인데 클러스터에서 어떻게 Cas를 사용할 수 있는것일까? 

답은 실제로 하나의 데이터에 대해서는 한 노드에서만 변경을 해서 가능하다이다. 데이터가 클러스터에서 어떻게 관리하는지 보면 알 수 있다.

## vBucket
CouchBase는 하나의 bucket을 1024개의 `vbucket`이라는 작은 단위로 쪼개서 vBucket들을 관리한다.(TODO: 사실 아직 bucket이란 단위의 개념을 공부를 하진 않았다..) 이 vBucket들은 클러스터의 노드 전체에 균등하게 분배가되어서 클러스터에서 부하가 쏠리지 않게 한다.

그런데 클러스터에서는 노드가 죽을일이 있으니 각각의 vBucket들이 replica를 가지고 있다. 그리고 이 replica들은 다른 노드들에 분포되게된다. 근데 이 replica들은 read 작업만 할 수 있고 write작업을 할 수 있는 녀석은 딱 하나이고 이것을 `active bucket`이라고한다. 

처음했던 질문으로 돌아와서 보면 한 document에 write 작업을 요청을 하면 active bucket에 접근을 하기때문에 한 노드에 대해서만 요청을 하게되고, 클러스터 내에서 동시에 여러 write요청이 와도 CAS를 통해 eventual consistency를 지킬 수 있게 되는것이다.

# multicluster에서 consistency

Coucbase는 XDCR이라고 하는 멀티 클러스터를 지원을 하는데 액티브 - 액티브 클러스터 관계라고 가정을 하였을때 동시에 다른 클러스터에서 하나의 document를 고치면 어떻게 될까?

XDCR coflict 정책에 따라 다르다고한다. 반영하거나 말거나 ㅎㅎ

---
참조
* https://docs.couchbase.com/java-sdk/current/howtos/concurrent-document-mutations.html
* https://docs.couchbase.com/server/current/learn/buckets-memory-and-storage/vbuckets.html
* https://docs.couchbase.com/server/current/learn/clusters-and-availability/xdcr-conflict-resolution.html