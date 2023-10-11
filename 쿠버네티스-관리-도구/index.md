# 쿠버네티스 관리 도구


쿠버네티스를 제대로 활용하려면 여러 대 서버를 클러스터로 구성해 사용해야 한다. 서버 자원에 쿠버네티스 클러스터를 직접 구성할 때 활용하는 도구는 Kubeadm, Kubespray가 있다.

## Kubeadm

**Kubeadm은 쿠버네티스에서 공식 제공하는 클러스터 생성/관리 도구이다.** 여러 대 서버를 쿠버네티스 클러스터로 손쉽게 구성할 수 있다. 

특징은 다음과 같다.

- 쿠버네티스에서 직접 제공하는 클러스터 생성 tool
- 클러스터 생성을 위해 필요한 기초 생성, 관리 명령어들이 포함된다.
- 클러스터를 관리하기 위한 것이므로 개별 node 에 대한 container runtime, kublet, cni 등은 알아서 설치가 필요하다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/b76f393f-d08b-4d0b-a9f3-bb5856b8427b)


Kubeadm에서 제공하는 클러스터 고가용성은 다음과 같은 구조이다.

![20231011_232919406](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/809220c3-2df7-4581-9bd6-deb7e8689662)

여러 대의 마스터 노드를 두고 그 앞에 로드밸런서를 두었다. 워커 노드들이 마스터 노드에 접근할 때는 로드밸런서를 거쳐 접근한다.


## Kubespray

**Kubespray는 상용 서비스에 적합한 보안성과 고가용성이 있는 쿠버네티스 클러스터를 배포하는 오픈소스 프로젝트이다.** Kubespray는 서버 환경 설정 자동화 도구인 앤서블 기반으로 개발되었다. 설정에 따라 사용자에게 맞는 다양한 형식으로 쿠버네티스 클러스터를 구성할 수 있으므로 온프레미스 환경에서 상용 서비스의 쿠버네티스 클러스터를 구성할 때 유용하다. 

특징은 다음과 같다.

- 쿠버네티스 클러스터 관리를 위한 오픈소스
- ansible 을 이용한 play book (인프라 담당자에게 익숙한 ansible 을 사용함)
- ansible 과 ssh를 사용하므로 대규모 쿠버네티스 클러스터를 관리하기에 적합하다.
- ansible 을 사용하므로 개별 worker node에 접속하지 않고 원격 설치 및 클러스터 구성이 가능하다.
- k8s 버전에 맞는 container runtime, cni, kubelet 에 대한 자동 설치를 제공한다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/5eda8082-21ef-4232-84a3-fdb11ca24c84)

Kubespray에서 제공하는 클러스터 고가용성은 다음과 같은 구조이다.

![20231011_232924114](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/a61ff08c-136e-41b9-bdf2-17a269e936f3)

Kubeadm처럼 별도의 로드밸런서를 사용하지 않고 노드 각각의 nginx가 리버스 프록시로 실행된다. 이 nginx-proxy가 전체 마스터 노드를 바라보는 구조이며, 그래서 쿠버네티스의 컴포넌트들은 직접 마스터 노드와 통신하지 않고 자신의 서버 안 nginx와 통신한다. 마스터 노드의 장애 감지는 헬스 체크를 이용해 nginx가 알아서 처리한다.

## 참고

쿠버네티스 입문, 동양북스

https://shonm.tistory.com/762

