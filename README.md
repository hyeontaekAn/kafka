# Strimzi Kafka란?
Strimzi란 쿠버네티스 환경에서 Kafka를 운영 관리할 수 있는 Operator이며, Apache Kafka를 쿠버네티스 환경에서의 프로세스를 단순화한다.

Strimzi Kafka는 아래와 같은 프로세스를 단순화할 수 있다.
* Kafka 클러스터 배포 및 실행
* Kafka 구성 요소 배포 및 실행
* Kafka에 대한 액세스 구성 및 보안
* Kafka 업그레이드
* 브로커 관리
* Topic 생성 및 관리

# Strimzi Kafka Architecture
Kafka의 아키텍처에는 다음이 포함된다. 

* Kafka의 서버 역할을 하는 Broker로 구성된 **Kafka Cluster**
* Kafka의 메타데이터 및 상태관리를 해주는 **Zookeeper**
* Kafka와 외부 데이터 연결을 위한 **Kafka Connect**
* Kafka Cluster를 미러링(보조 Cluster와 연결)하는 **Kafka MirrorMaker**
* 모니터링을 위해 Kafka Metric을 추출하는 **Kafka Exporter**
* Kafka Cluster와 HTTP 기반 통신을 할 수 있는 **Kakfa Bridge**

최소한 Kafka Cluster와 Zookeeper가 필요하지만 위 모든 구성 요소는 필수가 아니다.

예를 들어,  MirrorMaker와 Kafka Connect는 Kafka Cluster 없이 따로 배포가 가능하다.
<p align="center">
<img width="80%" src="https://strimzi.io/docs/operators/latest/images/overview/kafka-concepts-supporting-components.png"/>

## 설치 전 참고 사항
### 설치 버전
Operator 버전 별로 상이(아래 URL 참고)   
URL - https://strimzi.io/downloads/

본 문서에서는 아래와 같은 버전으로 진행한다.
|구분|버전|
|------|---|
|Kubernetes|1.26.3|
|Strimzi Operator|0.35.1|
|Kafka|3.4.0|
