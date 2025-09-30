# kubernetes-study

## 쿠버네티스 객체

- 쿠버네티스는 객체와 함께 동작
    - pods, deployements, services, volume 등

## Pod(파드)

- 쿠버네티스가 상호작용하는 가장 작은 단위
- 하나 또는 여러 컨테이너를 포함(보통 하나)
- 볼륨 등 리소스를 공유
- 기본적으로 클러스터 내부 IP를 가짐
- 내부에서 localhost로 통신 가능
- AWS ECS의 태스트와 유사
- 임시적(ephemeral)임 - 쿠버네티스에 의해 교체되거나 제거되면 도커 컨테이너 처럼 모든 리소스가 손실됨
- 쿠버네티스의 목적은 자동화된 배포 -> 파드를 직접 만들어 클러스터에 보내지 않고, 컨트롤러(특히, 디플로이먼트) 객체를 만들어 클러스터에 전송

## Deployment

- 보통, 파드를 직접 생성하여 워커노드에 배치하지 않고 디플로이먼트 객체를 생성
- 관리해야하는 파드 수, 컨테이너 수 제공
- 하나 이상의 파드를 제어
- 목표 상태에 도달하기 위해 필요한 모든 작업 수행
- 쿠버네티스에 의해 파드 객체가 생성되고, 컨테이너가 시작되며, 이 파드를 워커 노드에 배치
    - 원격 머신에 직접 파드를 배치할 필요가 없음
- 일시 중지, 삭제, 롤백 가능
- 동적으로 스케일 아웃될 수 있음(오토스케일도 가능)
- 쿠버네티스가 디플로이먼트 객체를 생성하고 클러스터에 전송하여 쿠버네티스가 이를 수행하도록 함

## 명령적(imperative)으로 배포

- 디플로이먼트 생성 명령어

```bash

kubectl create deploy --image ...

```

- 명령어 입력하면, 마스터 노드(컨트롤 플레인)가 클러스터에 필요한 모든 것을 생성
    - 스케줄러가 실행중인 파드들을 분석하여 가장 적합한 노드를 찾고 deployment 기반으로 파드 생성
    - 찾은 워커 노드가 kubelet 서비스를 얻고, 이 kubelet이 파드를 관리하고 컨테이너를 시작하고 파드를 모니터링
    - 이 부분에서 사용되는 컨테이너의 이미지는 디플로이먼트 생성할 때 지정한 이미지

## Service

- 파드를 다른 파드에게 또는 외부에 노출시킴
- 파드는 기본적으로 내부 IP를 가지지만 외부에서 접근 불가하며 파드가 교체될 때마다 바뀜
- IP가 항상 변경되기 때문에 파드를 찾는 것이 힘들기 때문에 서비스를 사용
- 서비스는 파드를 그룹화하고 IP를 공유
- 외부에서도 파드에 접근할 수 있게 해줌
- 서비스가 없으면 파드는 서로 통신하기 어렵고 외부에서 파드에 접근하는 것이 불가능

```bash

kubectl expose deployment <파드 이름> --type=ClusterIP/LoadBalancer --port=<포트> 

```

- `kubectl create service`로도 서비스 생성 가능

## 스케일링

```bash

kubectl scale deployment/<파드명> --replicas=<레플리카수>

```

- 레플리카 수 만큼 파드 생성

## Deployment 업데이트

```bash

kubectl set image deployment/<디플로이먼트명> <현재 컨테이너명>=<새 이미지명>

```

- 새 이미지로 변경(태그 변경 필요)

```bash

kubectl rollout status deployment/<디플로이먼트명> <현재 컨테이너명>=<새 이미지명>

```

- 결과 확인

## 롤백

```bash

kubectl rollout undo deployment/<디플로이먼트명>

```

- 원래 버전으로 돌아가기

```bash

kubectl rollout undo deployment/<디플로이먼트명> --to-revision=<돌아갈 버전>

```

- 특정 버전으로 돌아가기

## 선언적(declarative)으로 배포

- 리소스 정의 파일(yml) 파일에 구성 작성하고 파일 기반으로 배포
- 명령적 방식에서는 개별 명령어를 실행하여 동작을 트리거
- 선언적 방식에서는 구성 파일을 정의하여 원하는 상태로 변경
- 명령적 방식은 `docker run` 실행과 유사, 선언적 방식은 도커 컴포즈 사용하는 것과 유사

### Deployment 설정 파일 예제

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

- https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/

## Label과 Selector

- 디플로이먼트에서 spec에 selector 추가해줘야 함
- 디플로이먼트는 동적으로 파드를 생성하기 때문에 그 파드들을 관리하기 위해 셀렉터 필요
- selector에 matchLabels나 matchExpressions에 파드 레이블 키-밸류 쌍이나 규칙을 추가
- 파드는 레이블 키-밸류 쌍을 여러개 가질 수 있음
- selector는 template의 labels와 매칭되어야 함
- matchExpressions 사용하면 조건에 맞는 레이블 선택 가능
    - ex) `- {key: app, operattor: In, values [second-app, first-app]`
- 명령적 방식에서는 `-l` 옵션으로 레이블 선택 가능
    - ex) `kubectl delete deployments -l group=example`

## Service 설정 파일 예제

https://kubernetes.io/docs/concepts/services-networking/service/

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

- https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/

- 서비스의 셀렉터는 디플로이먼트의 파드 셀렉터 사용하면 됨

## 리소스 업데이트 및 삭제

- 선언적 방식에서는 설정 파일 변경 후 `kubectl apply -f <파일이름>` 명령어 실행하면 바로 업데이트됨
- `kubectl delete -f <파일이름>`로 해당 파일이 생성했던 리소스들을 삭제 가능

## 다중 구성 파일 vs. 단일 구성 파일

- 여러 구성 파일을 하나의 파일에서 `---`로 구분하면 하나로 합칠 수 있음

## Liveness Probes

- 쿠버네티스가 파드, 컨테이너 정상 여부 판단하는 방식 설정
- spec에 container에 livenessProbe로 설정
    - 요청 메서드(httpGet 등), 경로(path), 포트(port), 확인 빈도(periodSeconds), 초기 대기 시간(initialDelaySeconds) 등 설정 가능

## Volume

- 상태: 앱이 생성한, 삭제되어서는 안 되는 데이터(사용자 생성 데이터 - 데이터베이스, 파일에 저장, 앱에 의한 중간 결과 - 메모리나 임시 DB 테이블에 저장)
- 볼륨은 중요한 데이터가 손실되지 않게 하기 위한 수단
- 쿠버네티스에서는 컨테이너를 직접 실행시키지 않고 파드를 생성하여 컨테이너를 실행
- 컨테이너가 볼륨을 추가하도록 구성해야 함
- 쿠버네티스 볼륨은 도커 볼륨과 다르게 여러 가지 종류의 볼륨 타입 지원(로컬 볼륨, 클라우드 프로바이더 볼륨)
- 볼륨 수명 주기가 파드 수명 주기에 따라 다름
    - 볼륨은 컨테이너가 재시작해도 살아남음(볼륨은 파드 내부, 컨테이너 외부에 있기 때문)
    - 파드가 삭제되면 볼륨은 제거됨

### 도커 볼륨 vs. 쿠버네티스 볼륨

- 쿠버네티스 볼륨
    - 다양한 드라이버, 유형을 지원
    - 반드시 영구적인 것은 아님
    - 컨테이너 재시작 및 제거에도 살아남음
- 도커 볼륨
    - 드라이버, 유형 지원 없음
    - 수동으로 지우지 않으면 유지됨
    - 컨테이너 재시작 및 제거에도 살아남음

### 볼륨 유형

https://kubernetes.io/docs/concepts/storage/volumes/#volume-types

### emptyDir

- 파드가 시작될때마다 새로운 빈 디렉터리 생성
- 파드가 살아있는 동안은 이 디렉터리를 활성 상태로 유지하고 데이터를 채움
- 파드 제거 시 이 디렉터리는 제거됨
- 레플리카가 여러 개면 파드간 공유되지 않음
- 다른 파드로 전송되면 데이터는 손실됨

### hostPath

- 호스트의 경로가 파드에 연결됨
- 여러 파드가 호스트 머신의 동일한 경로를 공유
- 하나의 호스트 머신에 특정됨
    - 다른 노드에서 실행되는 여러 레플리카들은 동일한 데이터에 접근 불가

### CSI(Container Storage Interface)

- 다양한 클라우드 프로바이더와 사용 사례에 대해 더 많은 기본 유형을 추가하지 않기 위해 추가됨
- 이 인터페이스를 활용하면 누구나 드라이버 솔루션 구축 가능

### Persistent Volume(PV)

- 키 데이터와 같이 장기 데이터
- 애플리케이션 생성으로 손실되면 안되는 데이터를 위한 것
- 단순한 독립 스토리지 그 이상
- 파드로부터 볼륨을 분리하며 파드 수명 주기에서 완전히 벗어남
- 퍼시스턴트 볼륨은 파드와 노드에 대한 독립성을 갖게 되며, 클러스터 관리자로서 이 볼륨이 구성되는 방식에 대한 완전한 권한 갖게 됨
- 각각의 파드와 각각의 디플로이먼트 구성 파일 등을 여러 번 구성할 필요 없음
    - 한 번 정의 후, 여러 파드에서 사용 가능
- 파드와 독립적으로 볼륨 정의하고 중앙 위치에 저장
- 여러 퍼시스턴트 볼륨 유형 존재 - emptyDir, hostPath는 불가(hostPath는 싱글 노드 클러스터에서만 사용 가능)
- 설정
    - capacity
        - storage
    - volumeMode
        - FileSystem
        - Block
    - accessMode
        - ReadWriteOnce(하나의 노드에서만 읽기/쓰기 가능)
        - ReadOnlyMany(여러 노드에서 읽기만 가능)
        - ReadWriteMany(여러 노드에서 읽기/쓰기 가능)

### Persistent Volume Claim(PVC)

- 퍼시스턴트 볼륨 쓰려면 퍼시스턴트 볼륨 클레임(PVC) 필요
- 정적 볼륨 프로비저닝: 특정 퍼시스턴트 볼륨 요구
    - volumeName으로 간단히 지정 가능
- 동적 볼륨 프로비저닝: 원하는 크기나 설정에 맞춰서 알아서 볼륨 선택
- accessModes 지정 필요
- 구성 파일의 requests에 요구 사항 지정(용량 등)
    - 디플로이먼트에서 PV 사용 시 구성 파일에 persistentVolumeClaim 키에 claimName 명시
- 스토리지 클래스: 관리자에게 스토리지 관리 방법과 볼륨 구성 방법을 세부적으로 제어할 수 있게 해줌
    - 퍼시스턴트 볼륨 구성에 중요한 정보를 제공함
    - 퍼시스턴스 볼륨 spec에 storageClassName을 지정

### Volume vs. Persistent Volume

- 일반 볼륨
    - 일반 볼륨은 파드에 연결되며, 파드와 생명 주기를 함께 함
        - emptyDir은 재생성 시 빈 상태로 시작
        - hostPath나 다른 클라우드 프로바이더 유형은 데이터가 손실되지 않음
    - 파드와 동일한 파일에 구성이 정의됨
    - 파드 수가 많아지면 작업이 반복적이고 전역 수준에서 관리가 어려울 수 있음
- 영구 볼륨
    - 스탠드얼론 리소스(파드에 연결되지 않음)
    - PVC로 요구되며 스탠드얼론으로 생성됨
    - 한 번만 정의하면 여러 번 사용 가능

### 환경 변수

- 프로그래밍 언어에서 지원하는 방식으로 환경 변수 지정 후 구성 파일 내에서 env 키에 name, value 추가
    - Node.js라면 process.env.환경변수명으로 값 대체

### ConfigMap

## 네트워킹

### 내부 통신

- localhost

### 파드 간 통신

- service 이름에서 '-'을 '_'로 치환 후 + '.' + namespace

### 파드간 통신에 DNS

### nginx 리버스 프록시 사용하기