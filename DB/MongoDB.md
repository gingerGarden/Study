# MongoDB

### 1. __MongoDB의 개요__
1.1. MongoDB를 포함한 NoSQL의 장점
* 불필요한 Join의 최소화
* 유연성 있는 서버 구조 제공
* 비정형 데이터 구조로 설계 비용 감소
* Read/Write가 빠르며 빅데이터 처리 가능
> * 일반적인 관계영 DB가 빠른 경우도 있음
* 저렴한 비용으로 분산처리 및 병렬처리 가능
> * 비정형 데이터로 인해 관계형 DB보다 1.5배 용량을 더 많이 차지

<br>

1.2. NoSQL의 종류
* Key-Value: Redis, Memcached
* Column: Hbase, Casandra
* Document: MongoDB
* Graph: GraphDB

|Relation DB|NoSQL|
|----|----|
|Scalue-up: 서버 한대 중심으로 확장|Scale-out: 여러 대의 서버를 중심으로 확장|
|무결성|유연성|
|데이터 중복 제거|데이터 중복 허용|
|트랜잭션|빠른 쓰기, 읽기|

<br>

1.3. MongoDB의 특징
* MongoDB는 NoSQL로 분류되는 크로스 플랫폼 도큐먼트 지향 데이터베이스 시스템이다.
* 이름의 Mongo는 humongous의 줄인 말로, "엄청나게 큰 DB"라는 뜻이다.
* Json 타입의 Document 방식의 NoSQL이다.
> * Json 형식의 데이터 구조(MongoDB는 이를 BSON이라 부른다)
> * CRUD 위주의 다중 트랜잭션 처리 가능
> * Sharding(분산) / Replica(복제) 기능 제공
> * Memory Mapping 기술을 기반으로 빅데이터 처리 성능 탁월
* 똑같은 조건으로 설계 시, 기존 RDBMS보다 굉장히 빠름
> * 일반적인 RDBMS는 데이터의 중복을 제거하고, 무결성을 보장하기 위해 정규화하는데, 이러한 정규화로 인해 과도한 Join이 발생하여 성능 저하가 발생할 수 있음
> * MongoDB의 빠른 속도는 ACID를 포기한 댓가로 얻은 것으로, 데이터 일관성(Consistency)가 거의 필요 없고, Join 연산을 Embed로 대체할 수 있는 경우에는 MongoDB가 확실한 대안이 될 수 있음
> * 이러한 유연함은 개발자에게 스키마 팽창을 피해야한다는 부담을 의미하기도 하므로, 대규모 앱에서는 문서 구조에 대한 통제력을 유지하는 것이 중요
> * 그러나, 저장하는 데이터가 은행 데이터 같이 일관성(Consistency)이 매우 중요한 경우, MongoDB를 쓰기 매우 힘들다

<br>

1.4. MongoDB의 구조
* 관계형 DB와 MongoDB의 논리적 구조 비교
|Relational Database|MongoDB|
|----|----|
|Table|Collection|
|Row|Document|
|Column|Field|
|Primary key|Object_ID Field|
|Relationship|Embbeded & Link|
* MongoDB의 문서(Document)는 key-value의 집합으로 그 동작 방식은 객체와 매우 비슷함
* MongoDB의 구조
> * DB > Collection > Document
* __Document__: 가장 기본적인 데이터를 Document라 부르며, 이는 MySQL과 같은 RDBMS의 Row에 해당
> * Document의 최대 크기는 16MB
* __Collection__: Document의 집합으로, Collection은 RDBMS의 테이블에 해당함
> * Collection의 수가 많다고, 크게 성능 저하를 발생시키지는 않지만, 동일한 컬렉션 안에 많은 항목이 있다면, 성능에 영향을 줄 수 있음
> * 때문에 크기가 큰 컬렉션을 소모하기 쉬운 크기로 쪼갤 수 있는 방법을 고려해야함
* __DB__: Collection의 집합

![이미지 1](./images/MongoDB_instance.png)

<br>

1.5. MongoDB의 ID 필드
* 관계형 DB의 기본 키(Primary key) 개념은, 합성 ID 열, 즉 비즈니스 데이터와 관계되지 않고 생성된 값임
* MongoDB는 비슷한 목적으로 모든 Document에 *_id* 필드가 있음
* 개발자가 Document를 만들 때, ID를 별도로 제공하지 않는 경우, MongoDB 엔진이 UUID로 자동 생성
* *_id* 필드는 기본 키와 마찬가지로 자동으로 인덱싱되며, 고유해야함

<br>

1.6. MongoDB의 인덱싱
* MongoDB의 인덱싱은 관계형 DB의 인덱싱과 유사
* 문서의 필드에 대한 부가적인 데이터를 생성해 이 필드에 의존하는 조회의 속도를 높임
* MongoDB는 B-Tree 인덱스 사용
* 인덱스는 다음과 같은 구문으로 만들 수 있음
> * `db.pet.createIndex({name:1})
> * 매개변수의 정수는 인덱스가 오름차순(1)인지 내림차순(-1)인지 나타냄

<br>

1.7. MongoDB의 중첩 문서

![이미지 2](./images/MongoDB_document.png)

* MongoDB의 문서 지향 구조가 갖는 강력한 장점은 Document를 중첩할 수 있다는 것
* 예를 들어, 다음 예제와 같이 반려 동물 Document에 주소 정보를 저장할 또 다른 Field를 만드는 대신 중첩 Document를 만들 수 있음
```json
{
  "_id": "5cf0029caff5056591b0ce7d",
  "name": "Friar Tuck",
  "address": {
    "street": "Feline Lane",
    "city": "Big Sur",
    "state": "CA",
    "zip": "93920"
  },
  "type": "cat"
}
