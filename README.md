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