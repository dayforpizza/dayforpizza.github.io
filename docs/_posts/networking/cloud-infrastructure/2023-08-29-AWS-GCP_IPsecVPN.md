---
title: "AWS - GCP IPsec(VPN) 연결 방법"
excerpt: "AWS와 GCP를 IPsec(Site-to-Site) VPN 연결하려고 여러 글을 보며 시도했지만..."
header:
  teaser: /assets/posts/images/2023-10-03-10-19-16.png
categories:
  - Cloud Infrastructure
tags:
  - AWS
  - GCP
  - IPsec
  - VPN
---

# 개요
AWS와 GCP를 IPsec(Site-to-Site) VPN 연결하려고 여러 글을 보며 시도했지만 옛날 글들을 보고 따라하는 바람에 터널 상태가 계속 DOWN이라 계속 실패를 했다. 언제부터인지는 모르겠지만 BGP를 이용하여 VPN 연결하는게 기본이 되었나보다.

# 컨셉
- AWS와 GCP의 인스턴스(리눅스 서버) 간 사설 통신을 통한 SSH 연결
![](/assets/posts/images/2023-10-03-10-19-16.png)
- 해당 서버는 설정한 IP 외에는 접속 불가


# 준비
1. AWS와 GCP의 VPC 확인
 > - AWS Private Subnet(AZ A): 10.0.128.0/25
 > - GCP Private Subnet: 172.31.44.128/25
2. 인스턴스 생성
 > - AWS EC2 Instance(이하 EC2) 생성 (프리티어, Ubuntu 20.04)
 > - GCP Compute Engine Instance(이하 GCE) 2대 생성

_<font color="red">!! 개인 계정에서 테스트를 하는 거라면 꼭 삭제</font>_


# 작업 과정
![](/assets/posts/images/2023-10-03-10-19-36.png)


## 1. Cloud Router 생성
1) 하이브리드 연결 - Cloud Router - 라우터 만들기
- 리전: asia-northeast3
- Google ASN: 65000 (AWS 고객 게이트웨이 생성 시 사용)

![](/assets/posts/images/2023-10-03-10-19-44.png)


2) 생성된 라우터 확인

![](/assets/posts/images/2023-10-03-10-19-52.png)


## 2. VPN 생성
1) 하이브리드 연결 - VPN - VPN 연결 만들기
2) VPN 옵션 - 고가용성(HA) VPN 선택
- AWS와 VPN 연결 시 고가용성(HA)만 연결 가능

![](/assets/posts/images/2023-10-03-10-20-14.png)

3) Cloud HA VPN 게이트웨이 만들기
- VPN 게이트웨이 이름, 네트워크, 리전 설정

![](/assets/posts/images/2023-10-03-10-20-25.png)

4) VPN 터널 추가 항목에서 인터페이스(VPN 공인 IP 2개) 확인
- 현 과정에서 잠시 진행 보류

![](/assets/posts/images/2023-10-03-10-20-32.png)

## 3. AWS 고객 게이트웨이 생성
1) VPC - 가상 사설 네트워크(VPN) - 고객 게이트웨이(이하 CGW) 생성
- IP주소는 2.-4) 과정에서 GCP Cloud Router 생성 시 부여된 IP 입력 (2개 생성)
- BGP ASN: Google ASN과 동일한 값으로 입력 (_상이할 경우 VPN 터널 연결 불가_)

![](/assets/posts/images/2023-10-03-10-20-38.png)

- 고객 게이트웨이 생성 확인

![](/assets/posts/images/2023-10-03-10-20-48.png)


## 4. 가상 프라이빗 게이트웨이 생성
1) VPC - 가상 사설 네트워크(VPN) - 가상 프라이빗 게이트웨이(이하 VGW) 생성

![](/assets/posts/images/2023-10-03-10-20-56.png)

2) 생성된 VGW 선택 후 작업 - VPC에 연결

![](/assets/posts/images/2023-10-03-10-21-01.png)

3) 사용 가능한 VPC - 연결할 VPC 선택 후 연결

![](/assets/posts/images/2023-10-03-10-21-08.png)


## 5. Site-to-Site VPN 연결
1) VPN 연결 생성 - 세부 정보 설정
- 가상 프라이빗 게이트웨이 선택
- 고객 게이트웨이 ID 선택: cgw-for-gcp-1
- 라우팅 옵션: 동적(BGP 필요) 선택
- 로컬 IPv4 네트워크 CIDR: AWS IP 대역
- 원격 IPv4 네트워크 CIDR: GCP IP 대역

![](/assets/posts/images/2023-10-03-10-21-15.png)

2) 터널1, 터널2 옵션 - 별도 설정 안함

![](/assets/posts/images/2023-10-03-10-21-22.png)

3) 같은 과정을 한번 더 반복한다. *아래 내용만 변경
- 고객 게이트웨이 ID 선택 : cgw-for-gcp-2

4) 생성한 VPN 구성 다운로드
- VPN 2개 전부 다운로드

![](/assets/posts/images/2023-10-03-10-21-36.png)

- 공급 업체: Generic
- IKE 버전: ikev2

![](/assets/posts/images/2023-10-03-10-22-53.png)


## 6. GCP VPN Tunnel 생성
1) 하이브리드 연결 - VPN - CLOUD VPN 게이트웨이에서 VPN 터널 추가

![](/assets/posts/images/2023-10-03-10-23-03.png)

2) 피어 VPN 게이트웨이 생성
- AWS에서 Site-to-Site VPN을 2개 생성했고 1개 당 인터페이스 IP 2개가 부여된다.
- 구성 다운로드에서 받은 파일에 보면 Outside IP Addresses의 Virtual Private Gateway IP 주소를 적어준다. (총 4개)

![](/assets/posts/images/2023-10-03-10-23-14.png)

> - 구성 다운로드 파일
> ![](/assets/posts/images/2023-10-03-10-23-30.png)

3) 생성한 피어 VPN 게이트웨어 선택
4) VPN 터널 4개 만들기 선택
5) Cloud Router 선택

![](/assets/posts/images/2023-10-03-10-23-48.png)

6) VPN 터널 수정
- 이름, IKE 버전, IKE 사전 공유 키 설정

![](/assets/posts/images/2023-10-03-10-23-55.png)
- IKE 사전 공유 키는 AWS VPN의 구성 다운로드에서 확인

![](/assets/posts/images/2023-10-03-10-24-01.png)
- 나머지 터널 3개 설정 후 만들고 계속하기

![](/assets/posts/images/2023-10-03-10-24-08.png)


7) BGP 세션 구성
- 아래 표시된 "BGP 세션 구성" 선택

![](/assets/posts/images/2023-10-03-10-24-17.png)
- 피어 ASN 설정: VPN 구성 파일(BGP Configuration Options - Virtual Private  Gateway ASN 값)
> - VGW ASN 값: 64512 거의 고정이다.
- Allocate BGP IPv4 address 수동 선택
> - 구성 파일(Inside IP Addresses - Customer Gateway, Virtual Private Gateway 값)
> - /30 은 제외하고 입력

![](/assets/posts/images/2023-10-03-10-24-26.png)

- 4개 모두 구성 완료

![](/assets/posts/images/2023-10-03-10-24-36.png)

8) GCP VPN Tunnel 생성 완료

![](/assets/posts/images/2023-10-03-10-24-43.png)


## 7. 라우팅 확인
1) GCP 라우팅 설정 확인 (자동 추가됨)
- VPC 네트워크 - 경로 - 네트워크, 리전 선택 후 보기
- 필터 - 유형: 동적

![](/assets/posts/images/2023-10-03-10-24-51.png)

2) AWS 라우팅 전파 활성화
- VPC - 라우팅 테이블 - VPN 연결된 서브넷의 라우팅 테이블 선택
- 라우팅 전파 - 라우팅 전파 편집

![](/assets/posts/images/2023-10-03-10-24-57.png)
- 라우팅 전파 편집 - 가상 프라이빗 게이트웨이 - 전파 활성화

![](/assets/posts/images/2023-10-03-10-25-03.png)


## 8. 터널 상태 확인
1) AWS 터널
- 터널 세부 정보 - 상태 UP 확인

![](/assets/posts/images/2023-10-03-10-25-11.png)

2) GCP 터널
- CLOUD VPN 터널 - VPN 터널 상태, BGP 세션 상태: 설정됨 확인

![](/assets/posts/images/2023-10-03-10-25-17.png)

## 9. AWS Security Group 설정
> GCE 인스턴스 2대 확인

![](/assets/posts/images/2023-10-03-10-25-26.png)

1) GCP 대역 ICMP 허용, GCE 1대만 SSH 허용
- 172.31.44.130/32 SSH 허용

![](/assets/posts/images/2023-10-03-10-25-34.png)


## 10. Ping Test
1) GCE #1 (172.31.44.130) 
- Ping 정상

![](/assets/posts/images/2023-10-03-10-25-42.png)

2) GCE #2 (172.31.44.131) 
- Ping 정상

![](/assets/posts/images/2023-10-03-10-25-52.png)

## 11. SSH 접속 테스트
1) GCE #1 (172.31.44.130)
- 접속 성공

![](/assets/posts/images/2023-10-03-10-26-01.png)

2) GCE #2 (172.31.44.131) 
- 접속 실패

![](/assets/posts/images/2023-10-03-10-26-07.png)


# 마무리
> 테스트가 완료된 후 생성한 모든 자원을 꼭 삭제하길 바란다.


# 참고
https://blog.naver.com/PostView.naver?blogId=blueday9404&logNo=223055342609