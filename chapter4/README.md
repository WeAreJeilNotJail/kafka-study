# Chapter 4 **(카프카 상세 개념 설명)**

## 4.1 **토픽과 파티션**

### 적정 파티션 개수

> 토픽 생성 시 파티션 개수 고려사항
- 데이터 처리량
- 메시지 키 사용 여부
- 브로커, 컨슈머 영향도

> 컨슈머,프로듀서, 파티션 개수 공식

```프로듀서 전송 데이터량 < 컨퓨머 데이터 처리량 * 파티션 개수```

> 주의사항
- 브로커 당 파티션 개수 모니터링(프로세스당 열수 있는 파일 제한)
- 메세지 키 사용 여부(파티션 개수 바뀔 때 순서 보장 못함)

<br/>

* * *

<br/>

### 토픽 정리 정책(cleanup.policy)

#### **액티브 세그먼트란?**
- 데이터를 저장하기 위해 사용 중인 세그먼트
- segment.bytes옵션을 통해 세그먼트 크기 설정 가능

<br/>


> 토픽 삭제 정책(delete policy)
- 일반적인 설정
- 세그먼트 단위로 삭제
- 삭제 정책 실행 시점 - 시간(retention.ms) or 용량(retenthion.bytes)

> 토픽 압축 정책(compact policy)
- 메세지 키별로 오래된 데이터 삭제하는 정책
- 액티브 세그먼트를 제외한 나머지 세그먼트들에 한해 처리
- tail, head영역, clean, dirty 비율에 따라 처리 -> __min.cleanable.dirty.ratio__

### ISR(In-Sync-Replicas)
- 리더, 팔로워 파티션의 오프셋이 동일한 상태
- 복제 확인 주기 - __replica.lag.time.max.ms__
- ISR로 묶이지 못한 팔로워는 리더 선출 자격 X
- ISR아니어도 리더 선출하려면(데이터 유실 가능 O) - __unclean.leader.election.enable__

<br/>

* * *
* * *
<br/>

## 4.2 카프카 프로듀서

### acks 옵션
- 0,1,all, -1
- 프로듀서가 전송한 데이터 저장의 신뢰성
> akcs=0
- 저장되었는지 확인하지 않음
- 몇 번째 오프셋에 저장되었는지 확인 불가
- retries옵셧 무의미
- 전송 빠름

> acks=1
- 리더파티션에 저장되었는지 확인함
- 팔로워에 복제되지 않고 리더 장애 발생시 데이터 유실될 수 있음

> acks=all 또는 acks=-1
- 팔로워에도 정상 적재되었는지 확인
- **min.insync.replicas** 옵션에 따라 안정성 달라짐
- **min.insync.replicas** - ISR 최소 단위

<br/>

* * *

<br/>


### 멱등성 프로듀서
동일 데이터가 여러번 적재되어도 한번만 저장됨

> **enable.idempotence**

true인 경우 정확히 한번 전달 지원(멱등성)


<br/>

***
<br/>

### 트랜잭션 프로듀서
atomic하게 데이터 처리
- **enable.idempotenc** = true
- **transactional.id** = String
- **isolation.level** = read_committed


<br/>

***
***
<br/>

## 카프카 컨슈머

### 멀티 스레드 컨슈머
- 파티션 개수와 컨슈머 개수 동일하게 맞추는 것이 가장 좋은 방법

> 파티션이 n개인 경우
1. n개의 스레드를 가진 1개의 컨슈퍼 프로세스
2. 1개의 스레드를 가진 프로세스 n개

의 방법 등이 있다.

데이터 유실이 발생하지 않게 각 스레드간 영향이 없도록 조심해야함

> 카프카 컨슈머 멀티 워커 스레드 전략

> 카프카 컨슈머 멀티 스레드 전략

<br/>

***
***
<br/>

### 컨슈머 랙
- 최신 오프셋과 컨슈머 오프셋 간 차이
- 데이터양이 늘어나면 랙이 커짐
- 파티셧, 컨슈머 개수를 늘려 랙을 줄일 수 있음

> 컨슈머 랙을 확인하는 방법
1. 카프카 명령어
2. 컨슈머 어플리케이션에서 **metrics()**
3. 외부 모니터링 툴

> 카프카 명령어를 통한 랙 확인
- **kafka-consumer-groups.sh**
```shell
$ bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 \
  --group my-group --describe
```

> 컨슈머 metrics()메서드를 사용한 랙 확인
- records-lag-max
- records-lag
- records-lag-avg

> 외부 모니터링 툴을 사용
- 데이터독(Datadog)
- 컨플루언트 컨트롤 센터(Confluent Control Center)

<br/>

- 카프카 버로우

링크드인에서 공개한 오픈소스 컨슈머 랙 체크 툴
REST API를 통해 확인 가능


<br/>

***
***
<br/>

### 컨슈머 배포 프로세스

> 중단 배포
- 새로운 로직이 적용된 특정 오프셋 지점 나눌 수 있음
- 롤백도 간편(오프셋만 다시 바꾸면 되기 때문)

> 무중단 배포
- 블루/그린
- 롤링
- 카나리





































