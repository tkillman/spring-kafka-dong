# Spring-Kafka

* 참조
> https://www.popit.kr/kafka-%EC%9A%B4%EC%98%81%EC%9E%90%EA%B0%80-%EB%A7%90%ED%95%98%EB%8A%94-%EC%B2%98%EC%9D%8C-%EC%A0%91%ED%95%98%EB%8A%94-kafka/   <br/>
>
> https://godekdls.github.io/Apache%20Kafka/topic-configuration/
> 
* 주키퍼
> 주키퍼는 카프카의 노드 관리를 해주고, 토픽의 offset 정보등을 저장하기 위해 필요    
> 과반수 이상 살아 있으면 정상으로 동작합니다.  
> 
> ex) 주키퍼 3대일 경우 2대 이상, 주피커 5대일 경우 3대 이상

* 토픽
> 카프카에서 토픽은 데이터베이스의 table 정도의 개념   

* 파티션
> 메시지는 파티션별로 나뉘어 들어가고 consumer 는 파티션으로부터 메시지를 전달받으므로
> 순서를 보장받지 못한다.   
> 
> 순서를 보장받고 싶다면 하나의 파티션만 사용

* 컨슈머 그룹
> 컨슈머 그룹의 사용이유 ?    
> 
> server 를 여러 대 사용하여 안정성 확보   
> 
> offset 이라는 컨슈머그룹만의 책갈피를 만들어 어디까지 데이터를 가져온 것인지 구분 가능
> 
* 특징
> 카프카에서는 하나의 파티션에 컨슈머 그룹내 하나의 인스턴스만 접근이 가능

# config 정리
```java
@Bean
public NewTopic topic() {
    return TopicBuilder.name("thing2")
            .partitions(10)
            .replicas(3)
            .config(TopicConfig.COMPRESSION_TYPE_CONFIG, "zstd")
            .build();
}
```

* ## partitions   
  파티션 수 (한번 늘리면 줄일 수 없다.)

* ## replicas   
  replication factor 수 (수가 늘어날 수록 데이터 안전성이 보장)

## TopicConfig

* ### SEGMENT_BYTES_CONFIG = "segment.bytes";   
  기본값은 1GB. 세그먼트 파일이 지정된 크기에 도달하면 새로운 세그먼트 파일이 생성됨

* ### SEGMENT_MS_CONFIG = "segment.ms";
  새로운 세그먼트 파일이 생성된 이후 크기한도에 도달하지 않았더라도 다음 파일로 넘어갈 시간 주기를 정의

* ### SEGMENT_JITTER_MS_CONFIG = "segment.jitter.ms";
  세그먼트가 한꺼번에 롤링되지 않도록, 스케줄링한 세그먼트 롤링 시간에서 뺄 최대 랜덤 지터.

* ### SEGMENT_INDEX_BYTES_CONFIG = "segment.index.bytes";
   오프셋을 파일 위치에 매핑하는 인덱스의 크기 제어

* ### FLUSH_MESSAGES_INTERVAL_CONFIG = "flush.messages";
  로그를 기록할 데이터의 fsync 를 강제. 예를 들어 1로 설정하면 모든 메세지마다 fsync 실행. 5로 설정하면
메세지 5개마다 fsync 를 실행

* ### FLUSH_MS_CONFIG = "flush.ms";  
  로그에 기록할 데이터의 fsync 를 강제할 시간 간격 지정

* ### RETENTION_BYTES_CONFIG = "retention.bytes";
   "delete"보존 정책을 사용하는 경우, 공간 확보를 위해 오래된 로그 세그먼트를 폐기하기 전까지 파티션이 증가할 수 있는 최대 크기 제어

* ### RETENTION_MS_CONFIG = "retention.ms";
  "delete"보존 정책을 사용하는 경우 오래된 로그 세그먼트를 폐기하기 전까지 로그 최대 보존 시간 설정

* ### MAX_MESSAGE_BYTES_CONFIG = "max.message.bytes";
  카프카에서 허용할 레코드 배치의 최대 크기

* ### INDEX_INTERVAL_BYTES_CONFIG = "index.interval.bytes";
  오프셋 인덱스에 인덱스 엔트리를 추가하는 빈도 제어. 특별한 이유가 없으면 변경할 이유 없음.

* ### FILE_DELETE_DELAY_MS_CONFIG = "file.delete.delay.ms";
  파일 시스템에서 파일을 삭제하기 전 대기할 시간

* ### DELETE_RETENTION_MS_CONFIG = "delete.retention.ms";
  로그 컴팩션을 사용하는 토픽에서 삭제 툼스톤 마커를 보존하는 시간

* ### MIN_COMPACTION_LAG_MS_CONFIG = "min.compaction.lag.ms";
  메세지를 압축하지 않고 로그에 남겨둘 최소 시간. 압축하는 로그에만 적용된다.

* ### MAX_COMPACTION_LAG_MS_CONFIG = "max.compaction.lag.ms";
  메세지를 압축하지 않고 로그에 남겨둘 최대 시간. 압축하는 로그에만 적용된다.

* ### MIN_CLEANABLE_DIRTY_RATIO_CONFIG = "min.cleanable.dirty.ratio";
  로그 컴팩터가 로그 정리를 시도할 빈도를 제어

* ### CLEANUP_POLICY_CONFIG = "cleanup.policy";
  "delete" : 오래된 로그 세그먼트 보존 정책. 오래된 세그먼트 폐기, "compact" : 로그 컴팩션을 활성화

* ###  UNCLEAN_LEADER_ELECTION_ENABLE_CONFIG = "unclean.leader.election.enable";
  ISR 셋에 속하지 않은 레플리카를 리더로 선출할지 지정, 데이터가 유실될 수 있다.

* ### MIN_IN_SYNC_REPLICAS_CONFIG = "min.insync.replicas";
   쓰기를 승인해야 하는 최소 레플리카 수를 지정. 예로 replication factor 3으로 만들고 min.insync.replicas 2로 acks all 로 메세지 생성 시
레플리카 과반수 이상이 메세지를 받지 못하면서 프로듀서에 예외를 발생시킴을 보장할 수 있다.

* ### COMPRESSION_TYPE_CONFIG = "compression.type";
  압축타입 지정. 'uncompressed', 'gzip', 'snappy', 'lz4', 'zstd'

* ### PREALLOCATE_CONFIG = "preallocate";
  로그 세그먼트를 새로 만들 때 디스크에 파일을 미리 할당

* ### MESSAGE_TIMESTAMP_TYPE_CONFIG = "message.timestamp.type";
  메세지의 타임스탬프가 메세지 생성 시각인지 로그를 추가한 시각인지를 정의

* ### MESSAGE_TIMESTAMP_DIFFERENCE_MAX_MS_CONFIG = "message.timestamp.difference.max.ms";
  브로커가 메세지를 받은 시각과 메세지에 지정한 타임스탬프의 차이를 허용할 최대치

* ### MESSAGE_DOWNCONVERSION_ENABLE_CONFIG = "message.downconversion.enable";
  컨슈밍 요청에서 필요에 따라 메세지 포맷을 하위 버전으로 변환해줄지 여부를 제어. false이면 다운 컨버젼을 해주지 않음.












































