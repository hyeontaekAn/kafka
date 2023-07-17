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
Strimzi Kafka의 아키텍처에는 다음이 포함된다. 

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
Operator 버전마다 요구사항 상이(아래 URL 참고)   
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
Istio Proxy 사용 시 Zookeeper 기동이 안되는 오류가 있다. 

Zookeeper는 Pod IP에서만 수신 대기하기 때문에, Zookeeper에 대한 **Istio Injection을 제거**해야만 기동이 가능하다.   
(참고 URL - https://github.com/istio/istio/issues/19280)
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
**1. Strimzi Kafka Operator 다운로드** - `Kubernetes Master Node`에서 진행
- 현재 기준 Stable 버전으로 진행한다 - https://strimzi.io/downloads/
```sh
$ wget https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.35.1/strimzi-0.35.1.tar.gz
```

**2. 다운로드 받은 tar.gz 파일을 압축 해제한다.**
```bash
$ tar zxvf strimzi-0.35.1.tar.gz

strimzi-0.35.1
│  docs/
│  example/
│  install/
└─ CHANGELOG.md
```

**3. Strimzi Operator가 감시할 대상 Namespace를 생성한다.**
```bash
$ kubectl create namespace test-kafka
```
**4. Cluster Operator가 위 네임스페이스를 사용하도록 Strimzi 설치 파일을 수정한다.**
```bash
$ cd $STRIMZI_HOME

$ sed -i 's/namespace: .*/namespace: test-kafka/' install/cluster-operator/*RoleBinding*.yaml
```
**5. Cluster Operator(Cluster 관리)를 배포한다.**
```bash
$ kubectl create -f install/cluster-operator -n test-kafka
```
**6. 배포 상태를 확인한다.**
```bash
$ kubectl get deployments -n test-kafka
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
strimzi-cluster-operator     1/1     1            1           3d5h
```
> 위 상태와 같으면 정상적으로 배포된 상태이다.

**7. Kafka Cluster를 배포한다.**   
$STRIMZI_HOME/examples/kafka 디렉토리에 Sample YAML 파일들이 존재한다.
```bash
strimzi-0.35.1/examples/kafka
│  kafka-ephemeral-single.yaml
│  kafka-ephemeral.yaml
│  kafka-jbod.yaml
│  kafka-persistent-single.yaml
└─ kafka-persistent.yaml
```
* `kafka-ephemeral-single.yaml` - 3 Zookeeper, 1 Kafka 로 구성된 임시 클러스터(Pod 재기동 시 데이터 삭제)
* `kafka-ephemeral.yaml` - 3 Zookeeper, 3 Kafka 로 구성된 임시 클러스터
* `kafka-jbod.yaml` - 3 Zookeeper, 3 Kafka 로 구성된 영구 클러스터 (각각 여러 영구 볼륨 사용)
* `kafka-persistent-single.yaml` - 1 Zookeeper, 1 Kafka 로 구성된 영구 클러스터
* `kafka-persistent.yaml` - 3 Zookeeper, 3 Kafka 로 구성된 영구 클러스터 

본 문서에서는 3 Zookeeper, 3 Kafka로 구성된 임시 클러스터를 배포한다.
```bash
$ kubectl apply -f kafka-ephemeral.yaml
```
**8. 배포 상태를 확인한다.**
```bash
$ kubectl get pods -n test-kafka
NAME                                          READY   STATUS    RESTARTS        AGE
my-cluster-entity-operator-58f7457b4f-tkvmb   4/4     Running   0               6h19m
my-cluster-kafka-0                            2/2     Running   0               6h16m
my-cluster-kafka-1                            2/2     Running   0               3d7h
my-cluster-kafka-2                            2/2     Running   0               3d7h
my-cluster-zookeeper-0                        1/1     Running   0               6h16m
my-cluster-zookeeper-1                        1/1     Running   0               3d7h
my-cluster-zookeeper-2                        1/1     Running   0               3d7h
strimzi-cluster-operator-64d7d46fc-gtr8j      2/2     Running   0               6h19m
```
## 3. 설치 (Helm 사용)
**1. Helm 다운로드** - `Kubernetes Master Node`에서 진행   
참고 URL - https://helm.sh/ko/docs/intro/install/
```bash
$ helm version
version.BuildInfo{Version:"v3.11.2", GitCommit:"912ebc1cd10d38d340f048efaf0abda047c3468e", GitTreeState:"clean", GoVersion:"go1.18.10"}
```

**2. Strimzi Operator를 설치한다.**
```bash
$ helm repo add strimzi https://strimzi.io/charts/  # repo 추가
$ helm repo update                                  # repo 업데이트

$ helm repo list    # 추가 확인
NAME            URL
strimzi         https://strimzi.io/charts/
```

**3. Kubernetes Namespace를 생성한다.**
```bash
$ kubectl create namespace helm-strimzi  # kubectl create namespace <namespace명>
```

**4. Strimzi-Cluster-Operator를 배포한다.**
```bash
$ helm install helm-strimzi --namespace helm-strimzi strimzi/strimzi-kafka-operator
NAME: helm-strimzi
LAST DEPLOYED: Tue Jul  4 13:25:22 2023
NAMESPACE: helm-strimzi
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing strimzi-kafka-operator-0.35.1

To create a Kafka cluster refer to the following documentation.

https://strimzi.io/docs/operators/latest/deploying.html#deploying-cluster-operator-helm-chart-str
```

**5. 배포 확인**
```bash
$ helm list -A
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
helm-strimzi    helm-strimzi    1               2023-07-04 13:25:22.529077679 +0900 KST deployed        strimzi-kafka-operator-0.35.1   0.35.1
```
* CRD 배포 확인
```bash
$ kubectl get crd -A |grep kafka
kafkabridges.kafka.strimzi.io                         2023-07-04T04:25:20Z
kafkaconnectors.kafka.strimzi.io                      2023-07-04T04:25:20Z
kafkaconnects.kafka.strimzi.io                        2023-07-04T04:25:20Z
kafkamirrormaker2s.kafka.strimzi.io                   2023-07-04T04:25:20Z
kafkamirrormakers.kafka.strimzi.io                    2023-07-04T04:25:20Z
kafkarebalances.kafka.strimzi.io                      2023-07-04T04:25:20Z
kafkas.kafka.strimzi.io                               2023-07-04T04:25:20Z
kafkatopics.kafka.strimzi.io                          2023-07-04T04:25:20Z
kafkausers.kafka.strimzi.io                           2023-07-04T04:25:20Z
```
**6. Kafka Cluster를 배포한다.** 

해당 단계 내용부터는 위 yaml등록 방식과 동일하다. 아래 URL을 통해 Kafka Cluster YAML파일을 볼 수 있다.   
URL - https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples/kafka

## 4. 테스트
**1. Topic 생성** - `topic.yaml`   
```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: my-topic  # Topic 이름
  labels:
    strimzi.io/cluster: my-cluster  # 배포한 Kafka Cluster 이름 (Default=my-cluster)
spec:
  partitions: 3  # Topic의 파티션 수
  replicas: 3    # Topic의 복제 수
  config:
    retention.ms: 7200000
    segment.bytes: 1073741824
```
```bash
$ kubectl apply -f topic.yaml -n <strimzi 배포 네임스페이스 명>
```

**2. Topic 생성 확인**
```bash
$ kubectl get kafkatopics -n <strimzi 배포 네임스페이스 명>
NAME        CLUSTER      PARTITIONS   REPLICATION FACTOR   READY
test        my-cluster   3            3                    True
...
```

**3. Kafka Pub/Sub 테스트**   
Strimzi Kafka 배포 시 Service는 ClusterIP로 같이 배포된다.   
테스트 방법은 아래의 예시를 참고한다.
1. `Kubernetes Master Node`에 Apache Kafka를 직접 설치하여 ClusterIP로 연결
2. Apache Kafka 바이너리를 이미지로 직접 빌드하여 `Kubernetes Pod` 배포 후 클라이언트 사용 - `kafka-console-producer.sh`
3. bitnami/kafka 이미지를 이용하여 `Kubernetes Pod` 배포 후 클라이언트 사용 - `kafka-console-consumer.sh`

본 문서에서는 **3. bitnami/kafka 이미지**를 사용하여 테스트를 진행한다.

**3.1. Kafka Client Pod 배포** - `Client.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: strimzi-test    # Pod 이름
  labels:
    app: myclient       # Label 지정 (옵션)
spec:
  containers:
  - name: test                # Container 이름
    image: bitnami/kafka:3.4  # 사용 이미지 (bitnami/kafka v3.4)
    command: ["tail"]
    args: ["-f", "/dev/null"]
```
```bash
$ kubectl apply -f Client.yaml -n app  # app = namespace 이름
```

**3.2. 배포 확인**
```bash
$ kubectl get pod -n app | grep strimzi-test
NAME           READY   STATUS    RESTARTS   AGE
strimzi-test   2/2     Running   0          173m
```

**3.3. 연결된 Service 확인**
```bash
$ kubectl get svc -n <Strimzi 배포 네임스페이스 명>
NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
my-cluster-kafka-bootstrap    ClusterIP   10.109.64.220    <none>        9091/TCP,9092/TCP,9093/TCP            3h48m
```
Strimzi Kafka에 연결된 서비스 이름은 다음과 같다.
* 서비스 이름 : `<배포된 Kafka 클러스터 이름>-kafka-bootstrap`

> 9091포트 - 브로커 간 통신 | 9092포트 - Client와 PLAINTEXT 통신 | 9093포트 - Client와 TLS 통신

**3.4. Pub/Sub 테스트**   
Kubernetes에서 사용하는 도메인은 다음과 같다.
* 도메인 : `<서비스 이름>.<네임스페이스 이름>.svc`

```bash
# Producer 연결
# kubectl exec -it -n <Kafka Client Pod 이름> -- kafka-console-producer.sh --bootstrap-server <Strimzi Kafka 도메인:포트> --topic <토픽 이름>
$ kubectl exec -it -n app strimzi-test -- kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap.helm-strimzi.svc:9092 --topic test
```
```bash
# Consumer 연결
# kubectl exec -it -n <Kafka Client Pod 이름> -- kafka-console-consumer.sh --bootstrap-server <Strimzi Kafka 도메인:포트> --topic test
$ kubectl exec -it -n app strimzi-test -- kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap.helm-strimzi.svc:9092 --topic test
```

## 5. 출처
Strimzi Kafka - https://strimzi.io/

