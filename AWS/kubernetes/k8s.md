## docker
### vm -> container 이유
1. 관리 point가 다양... -> 묶어서 동일한 환경을 재구성 할 수 있도록 보장
2. docker 이미지를 전달해서 동일한 환경을 어디서든 사용 할 수 있음

## 처음에 vm을 사용함
1. 기획자, qa등 테스트 환경을 구성해야 됨 -> 관리하기 불편 -> vm 사용(격리 환경 구성)
2. 서버안에 필요할때 마다 vm을 설치 -> 물리 이미지를 관리할 필요가 없어져 관리가 용이해짐
3. 버젼 마다 vm을 설치해야 되서 리소스 낭비

## vm -> container
1. docker로 띄우면 vm에서 잡고 있는 리소스 저하가 적고 관리가 용이해짐


### docker 구성
1. 열광받은 이유중에 하나는 이미지 구성이 잘 되어 있음
2. docker 이미지를 바탕으로 -> 인스턴스 구성
3. 이미지가 만들어 질때 바닥부터 차곡차곡 쌓여짐
4. layering -> 같은 이미지는 공유 (base image) -> 구성하는데 시간이 줄어듬
5. 자주 바뀌는 이미지는 최대한 위쪽에 올려주어야 caching을 활용할 수 있음

### docker hub
1. official image 기본적인 이미지들은 그대로 가져다 사용 가능
 
### docker만 가지고 많은 서버를 관리하기 힘듬 -> continaer orchestration
1. 많은서버에 직접 접속해서 docker run?
2. 많은서버 중 몇대가 죽음 -> 찾기 힘듬
3. 그래서 나온게 orchestration

### container orchestration
1. 서버가 죽었을 경우 지속적으로 서비스가 유지되게 끔 (가용성)
2. 노드가 천개가 있을때 어떤 노드가 뭔지 관리하기 힘듬 -> orchastrator가 어떤서버에 무슨일이 있는지 추적
3. 어디다 띄울지, auto scaling
4. 그래서 저장소가 필요 -> clustering 관리를 위해 정보를 담고 있는 저장소가 필요

*kubernetes(K8s)
1. google -> cncf재단 -> open source

### k8s components
1. control plan 하나
2. worker node 여러개
3. 1에서 다수의 2를 관리


### k8s objects
## kubectl
1. 정형화된 툴
2. ~/.kube/config 안에 kubectl을 통해서 어떤 클러스터, 어떤 유저에 접속을 할지에 대한 정보를 저장

## pods
1. docker와 다른 차별점 중 하나
2. k8s에서 최소 실행 단위
3. 하나 이상의 컨테이너들이 들어있음
4. 하나 혹은 다수의 컨테이너 묶음
5. 하나의 pods에서는 컨테이너들끼리 볼륨을 share
6. ip 공유 docker는 컨테이너 마다 ip가 생기지만, k8s pods는 pods 당 ip가 생김
7. 컨테이너들 끼리는 localhost로 통신, pods는 동일한 포트를 사용 할 수 없음


## labels
1. key/value pairs
2. 자체에는 기능이 없지만 tags 처럼 사용 가능 

## replicaSets
1. pods에서 발전된 개념
2. pods를 n개 띄우는 걸 유지

## deployment
1. replicaSets에서 확장됨
2. replicaSets + updates
3. k8s는 yaml로 updates된 버젼으로 설정을 하고 적용하면
4. updates 가능(rolling update), 따라서 보통 deployments 사용

## untimely died
1. 5개중 2개의 서버가 죽음
2. 근데 알고보니 스위치 이상이라, 잠시후 다시 살아남
3. k8s는 살아난걸 모름
4. balnder가 pods 개수 안맞는걸 확인하고 죽임
5. 죽이면 다시 k8s가 살림

## services
1. 여러개의 pods들을 묶어서 하나의 pods 처럼 관리해줌
2. AWS ELB라 생각하면 되고 L4 Switch 처럼 동작한다고 생각하면 됨 (load balancer)
3. EC2를 LB에 등록 -> 여기서 labels을 이용함 (selector)
4. IP를 대상으로 loadBalancing

## deployment-blue/green
1. 처음에 deployment -> replication set (version = 1) (blue)
2. version 변경해서 deployment -> replication set (version = 2) (green)
3. service version 1-> 2 하면 blue-> green

## service - clusterIP
1. k8s 에서 사용하는 local 안에서만 접속이됨

## service - nodePort
1. 인터넷에서 사용 가능
2. 모든 vm의 포트를 똑같이 열어둠
3. 어느 IP의 포트로 붙어도 실제로는 똑같은 서비스에 붙음
4. 실제로 k8s는 public cloud에서 사용됨
5. Users Ports 30000 ~ 32767

## service - loadBalancer
1. nodePort 타입을 사용
2. load balancer는 대상이 IP, Port 등록
3. 가상머신이 몇대가 떠있든 상관없음
4. load balancer에 등록되는 애들은 nodePort


                             
## service - ingress
1. ingress가 domain을(http/https routes) 대상으로 쪼개주고
2. 서비스에 연결

### configMap and secret
1. pods마다 일일이 config 하는게 아니라 configmap으로 만들어두고
2. pods로 전달, 환경변수나 볼륨으로 선택해서 만들수 있음
3. 똑같은 개념에서 암호화해서 들어가는게 secret
4. secret default는 base64 encoding
5. 복잡한 암호화가 필요하면 kms나 다른방법 이용

### stateful
## stateful properties
1. network identifier (이름고정)
2. persistent storage (영구적인 데이터 저장 필요)
3. .. 등등

## statefulSet

## storage
1. 기본적으로 container를 임시저장
2. k8s에서 persistent는 network volume을 말함

## lifecycle of a storage volume

#### k8s statful을 이용하는건 docker를 다른데서 똑같이 띄운다는 장점..