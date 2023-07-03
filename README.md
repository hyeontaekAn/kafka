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

## 1. 설치 전 참고 사항
### 설치 버전
Operator 버전 별로 상이(아래 URL 참고)   
URL - https://strimzi.io/downloads/

본 문서에서는 아래와 같은 버전으로 진행한다.
|구분|버전|
|------|---|
|Kubernetes|1.26.3|
|Strimzi Operator|0.35.1|
|Kafka|3.4.0|
|helm|3.11.2|

### JAVA 버전
Kafka Cluster의 Pod에 설치되는 JAVA 버전은 17이다.

### Kubernetes 설치
Strimzi는 쿠버네티스 환경을 위한 Operator이다. 

쿠버네티스 설치는 **필수**이며, 본 가이드에서는 **1 Master, 3 Worker Node**로 진행한다.


### Istio 사용 여부
Istio Proxy 사용 시 기동이 안되는 오류가 있다. 

이는 Strimzi에서 제공하는 Zookeeper의 의도된 방향이며, Zookeeper에 대한 **Istio Injection을 제거**해야만 기동이 가능하다.
```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
...
  zookeeper:
    replicas: 3
    template:
      pod:
        metadata:
          labels:
            sidecar.istio.io/inject: "false" # 해당 문구 추가
```

### 설치 방식
본 문서는 **yaml 파일**을 apply하여 Strimzi를 설치하는 방식과 **helm**으로 설치하는 방식을 설명한다. 

**Minikube, Kubernetes Kind, Docker Desktop**으로 설치하는 방법은 아래의 URL을 참고한다.   
URL - https://strimzi.io/quickstarts/

## 2. 설치 (yaml 등록 방식)
1. Strimzi Kafka Operator 다운로드 - `Kubernetes Master Node`에서 진행
- 현재 기준 Stable 버전으로 진행한다 - https://strimzi.io/downloads/
```sh
wget https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.35.1/strimzi-0.35.1.tar.gz
tar zxvf strimzi-0.35.1.tar.gz
```

2. 다운로드 받은 tar.gz 파일을 압축 해제한다.
```bash
strimzi-0.35.1
│  docs/
│  example/
│  install/
└─ CHANGELOG.md
```

3. Strimzi Operator가 감시할 대상 Namespace를 생성한다.
```bash
kubectl create namespace test-kafka
```
4. Cluster Operator가 위 네임스페이스를 사용하도록 Strimzi 설치 파일을 수정한다.
```bash
cd $STRIMZI_HOME

sed -i 's/namespace: .*/namespace: test-kafka/' install/cluster-operator/*RoleBinding*.yaml
```
5. Cluster Operator를 배포한다.
```bash
kubectl create -f install/cluster-operator -n test-kafka
```
6. 
