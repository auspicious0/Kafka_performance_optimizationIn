# Kafka_performance_optimizationIn

## 목표:
```
데이터 처리량을 증가
데이터 처리 속도 증가 
메시지 유실을 최소화 하는 것
서버 안정성을 최대화 하는 것.
```
## 관련 설정

### 1.	Producer 
```
batch.size: 값 증가시 한 번에 많은 양을 보내며 전송 횟수 감소 서버 부하 감소

linger.ms: 보내기 전 메시지를 기다리는 시간을 정의 배치 사이즈에 도달하면 이 옵션과 상관없이 메시지 전송. 따라서 batch size 수정시 batch size에 시간을 맞춰야 함.
compression.type: 압축 타입을 지정하여 데이터를 압축해서 보낼 수 있다. (옵션:none, gzip, snappy, lz4)
buffer.memory: 서버로 데이터를 보내기 위해 잠시 대기할 수 있는 전체 메모리 바이트
```
acks: 프로두서가 카프카 토픽의 리더에게 메시지를 보낸 후 요청을 완료하기 전 ack의 수
```
ack=0 일 때, 프로두서는 서버로부터 어떠한 ack도 기다리지 않는다. 따라서 속도와 처리량이 높지만 서버가 데이터를 받았는지 보장하지 않고 전송 실패에 대한 결과를 알 수 없다. 
ack=1 일 때, 리더는 데이터를 기록하지만 모든 팔로워는 확인하지 않는다. 일부 데이터 손실이 발생할 수 있다.
acks=all(-1): 리더와 팔로워 모두 기다립니다.
```


```
retries: 일시적인 오류로 인해 전송에 실패한 데이터를 다시 보내는 횟수
```
### 2.	Broker
```
default.replication.factor: 복제 요소를 설정하는 옵션 증가 시 분산처리 증가

num.replica.fetchers: 브로커의 메시지 복제하는 스레드 수 지정 값 늘어나면 병렬처리 증가 가능
auto.create.topics.enable: 토픽 자동 작성(default가 트루임) 토픽이 계속 늘어날 수 있음 false로 하는 것이 성능 효과적
min.insync.replicas: 최소 리플리케이션 펙터 -> 브로커가 3대 있고 min.insync.replicas가 3이라면 1개만 고장나도 메세지 보낼 수 없는 상태, 2로 설정되어 있다면 메시지 보낼 수 있음 감당 가능 낮을수록 유리. but 메시지 유실 가능성 증가

unclean.leader.election.enable: 데이터 유실이 발생하더라도 kafka 서버가 중지되는 상황을 막는 설정 -> isp(in-sync replica)가 아닌 osr(out of sync replica)를 가지고 있는 broker를 leader로 선출할 수 있도록 선출하는 옵션.
 

broker.rack: 한쪽으로 task가 쏠리지 않게 하는 시스템 -> 분산처리의 핵심이다. 어떻게 사용하는지 잘 모르겠으나 현재까지 살펴본 바로는 같은 서버에 있는 broker마다 고유의 아이디를 생성하는 것 같다.
 
log.flush.interval.messages : 디스크로 보내기 전 메모리에 보유할 메시지의 개수 -> 적을수록 유리할까? 잘

log.flush.interval.ms: 메시지가 플러시 되기 전에 로그에서 누적할 수 있는 최대 시간


num.recovery.threads.per.data.dir: 각 데이터 디렉토리에 대한 로그 복구에 사용되는 스레드의 최대 수
```

		로그에 이슈가 있었기 때문에 일단 로그 및 디스크 관련된 변수를 모아봄.

### 3.	CONSUMER
fetch.min.bytes: 한 번에 가져올 수 있는 최소 데이터 사이즈 클수록 유리. 
fetch.max.wait.ms: 데이터를 가져오는 최소 시간 클수록 유리 -> 속도 측면에서

### 4. 퍼포먼스 도구 
./kafka-producer-perf-test.sh --topic test --num-records 20000 --record-size 800 --throughput -1 --producer-props bootstrap.servers=192.168.1.93:9092,192.168.1.93:9091,192.168.1.93:9093

