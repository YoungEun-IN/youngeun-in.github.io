# 사설IP, NAT, CIDR


## 사설 IP(Private IP)

**사설 IP(Private IP)는 한정된 IP 주소를 최대한 활용하기 위해 IP주소를 분할하고자 만든 개념이다.** IPV4 기준으로 최대 IP 개수는 약 43억개이다.

사설망 내부에는 외부 인터넷 망으로 통신이 불가능한 사설 IP가 구성된다. 그리고 외부로 통신할 때는 통신 가능한 공인 IP를 사용한다. 보통 하나의 망은 사설 IP를 부여받은 기기들과 NAT 기능을 갖춘 Gateway로 구성한다.

## NAT(Network Address Translation)

**NAT(Network Address Translation)는 사설 IP가 공용 IP로 통신할 수 있도록 주소를 변환해주는 방법이다.**

- Dynamic NAT : 1개의 사설 IP를 가용 가능한 공인 IP로 연결하는 방식
  - 공인 IP 그룹(NAT Pool)에서 현재 사용 가능한 IP를 가져와서 연결한다.
- Static NAT : 하나의 사설 IP를 고정된 하나의 공인 IP로 연결하는 방식. Public IP와 Private IP의 숫자가 같다.
  - AWS Internat Gateway가 사용하는 방식이다.
- PAT(Port Address Translation) : 많은 사설 IP를 하나의 공인 IP로 연결. 하나의 Public IP가 사설망을 대표하고 각 통신 시에는 port를 다르게 한다.
  - 대부분의 사설 네트워크가 사용하는 방식이다.

## CIDR(Classless Inter Domain Routiong)

**CIDR(Classless Inter Domain Routiong)는 여러 개의 사설망을 구축하기 위해 망을 나누고 IP를 묶는 방식을 말한다.**

CIDR 은 네트워크 주소와 호스트 주소로 구성되며, 각 호스트 주소 숫자 만큼의 IP를 가진 네트워크 망 형성 가능하다.

CIDR은 A.B.C.D/E 형식으로 구성된다. A.B.C.D는 네트워크 주소+호스트 주소를 표시하며, E는 네트워크 주소가 몇 bit인지 표시한다. 호스트 주소 비트 만큼 IP 주소를 보유 가능하다.

예를 들어 CIDR이 192.168.2.0/24이라면, 네트워크 비트는 24이고 호스트 주소는 32-24=8이다. 즉 2^8 = 256개의 IP 주소 보유 가능하다.

### 서브넷

**서브넷은 네트워크 안의 네트워크를 말한다.** 일정 IP주소의 범위를 보유하며, 큰 네트워크에 부여된 IP 범위를 조금씩 잘라 작은 단위로 나눈 후 각 서브넷에 할당하는데 이때 CIDR이 사용된다.

## 참고

https://www.youtube.com/watch?v=3VXLD0-Iq8A

