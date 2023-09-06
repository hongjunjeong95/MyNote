# End point 생성하기

  NAT Gateway를 생성한 것으로 충분히 외부에 접근해서 패키지를 다운받거나 서비스를 이용할 수 있다. 하지만, 트래픽이 노출되기 때문에 보안상 좋지않다. 그래서 외부에 있는 AWS 서비스를 이용할 때는 엔드포인트를 이용해서 제한적으로 외부와의 네트워크 연결을 허용해야한다.

## 실습 목표

* IAM 역할 생성
* S3에 접근하는 엔드포인트 확인을 위한 public & private ec2 생성
* 엔드포인트 생성
* 라우팅 테이블 편집
* priavte instance로 aws s3 접근



## IAM 역할 생성 for S3
1. "AWS Console" -> "IAM" -> 좌측 "액세스 관리" -> "역할"

   <img src="https://user-images.githubusercontent.com/33750210/137651608-763f26af-c1a1-4bf3-a725-e16954d3a418.png" alt="image" style="zoom:80%;" />

2. **역할 만들기** 클릭
   
    ![image](https://user-images.githubusercontent.com/33750210/137651681-e86d30af-274c-4d07-80b8-da99880c718b.png)
    
3. 사용 사례 선택

    <img src="https://user-images.githubusercontent.com/33750210/137651807-f296218b-17e4-456c-8775-a6fc81af2ea2.png" alt="image" style="zoom:55%;" />

    **EC2** 선택 => **다음: 권한**
    
    
    
3. 권한 정책 연결

    <img src="https://user-images.githubusercontent.com/33750210/137652273-d794a252-5d3c-4b3d-991f-e95b81ad81cc.png" alt="image" style="zoom:67%;" />

    1. 정책 필터 : **s3** 검색
    2. **AmazonS3FullAccuess** 선택
    3. **다음: 태그** 클릭
    
    
    
3. 태그 추가(선택 사항)

    <img src="https://user-images.githubusercontent.com/33750210/137652455-4ef42889-4802-44c8-99d0-80601efc7a57.png" alt="image" style="zoom:55%;" />

    * Name : s3_fullaccess
    * **다음: 검토** 클릭
    
    
    
6. 검토

    ![image](https://user-images.githubusercontent.com/33750210/137652529-e5ceb984-dd1f-498b-a57c-bb1c2c832eb4.png)

    1. 역할 이름 : s3_fullaccess
    2. 역할 설명 : s3_fullaccess
    3. **역할 만들기** 클릭
       * 역할이 생성됨.
    
    

## Public EC2 생성

 1. 단계1: AMI 선택 : Ubuntu Server 20.04 LTS

 2. 단계2 : 인스턴스 유형 선택 : t2.micro(프리미어 티어 사용 가능)

 3. 단계3 : 인스턴스 세부 정보 구성

    * 네트워크 : insomenia-vpc
    * 서브넷 : insomenia-public-subnet
    * 퍼블릭 IP 자동 할당 : 서브넷 사용 설정(활성화)
    * IAM 역할 : x
      * private ec2에서 접근할 것이고 public ec2는 단순히 private ec2에 접근하는 bastion host이기 때문에 s3_fullaccess IAM 권한이 필요가 없다.

 4. 단계 4 : 스토리지 추가 : 범용 SSD(gp2)

 5. 단계 5 : 태그 추가

    * 키 - 값 : Name - insomenia-public-ec2

 6. 단계 6 : 보안 그룹 생성

    ![image](https://user-images.githubusercontent.com/33750210/137618601-114c31ea-c7bf-4c9d-a1ed-c29bab0720a1.png)

    * 유형 : 모든 ICMP - IPv4
    * 프로토콜 : ICMP
    * 포트 범위 : 0-65535
    * 소스 : 위치 무관
    * **검토 및 시작** 클릭

7. 단계 7: 인스턴스 검토 시작 - 기존 키 페어 선택: "public" 이름이 들어간 기존 키 페어 선택.

    * 키 페어가 없다면 새로 생성



## Private EC2 생성

 1. 단계1: AMI 선택 : Ubuntu Server 20.04 LTS

 2. 단계2 : 인스턴스 유형 선택 : t2.micro(프리미어 티어 사용 가능)

 3. 단계3 : 인스턴스 세부 정보 구성

    * 네트워크 : insomenia-vpc
    * 서브넷 : insomenia-private-subnet
    * 퍼블릭 IP 자동 할당 : 서브넷 사용 설정(비활성화)
    * IAM 역할 : s3_fullaccess

    ![image](https://user-images.githubusercontent.com/33750210/137653039-93e463ff-1355-4522-95c9-6f15586cee5a.png)

 4. 단계 4 : 스토리지 추가 : 범용 SSD(gp2)

 5. 단계 5 : 태그 추가

    * 키 - 값 : Name - insomenia-private-ec2

 6. 단계 6 : 보안 그룹 생성

    ![image](https://user-images.githubusercontent.com/33750210/137619088-7ceb55b4-dd9a-4752-b3ac-fa8a002c10e6.png)
    * 유형 : 모든 ICMP - IPv4
    * 프로토콜 : ICMP
    * 포트 범위 : 0-65535
    * 소스 : public-sg
      * Private EC2는 관계자 외 외부인에게 노출되어서는 안 되고 접근을 허용해서도 안 되기 때문에 사용자가 지정한 sg가 적용된 host에서만 접근을 허용해야한다.
      * 검색할 때 insomenia-public-ec2에 적용한 보안 그룹 이름을 검색하면 결과 값을 보여준다.
    *  **검토 및 시작** 클릭
    
7. 단계 7: 인스턴스 검토 시작 - 기존 키 페어 선택: "private" 이름이 들어간 기존 키 페어 선택.

   * 키 페어가 없다면 새로 생성



## VPC 엔드포인트 생성

1. "AWS Console" -> "VPC" -> 좌측 "가상 프라이빗 클라우드" -> "엔드포인트"로 가기

   <img src="https://user-images.githubusercontent.com/33750210/137657222-6b01b39e-e8f6-4578-baab-cae20cb65734.png" alt="image" style="zoom:67%;" />

   

2. "엔드포인트 생성" 클릭
   
    ![image](https://user-images.githubusercontent.com/33750210/137657266-9b0b6d67-1a64-49f3-90a6-1144ec53f8a5.png)

   

3. 엔드포인트 설정

   ![image](https://user-images.githubusercontent.com/33750210/137657710-62ce7a00-0883-46ac-882c-e8ec5ec2e2e1.png)

   * 서비스 이름 : com.amazoneaws ap-northeast-2.s3
     * 같은 이름이 두 개가 존재하는데, 유형이 **Gateway**인 것을 선택해야 한다.
     
     ![image](https://user-images.githubusercontent.com/33750210/137657641-db7de483-2e9c-4ef3-b25e-383b626a9560.png) 

   * VPC : insomenia-vpc

   * 라우팅 테이블 : private-subnet routing table(insomenia-private-subnet) 선택

   

   ![image](https://user-images.githubusercontent.com/33750210/137657759-d64e460f-8d80-4ca4-92b9-c57911d34e83.png)

   * **엔드포인트 생성**

   

4. 엔드포인트가 추가된 private 라우팅 테이블 확인
   
   ![image](https://user-images.githubusercontent.com/33750210/137658123-9c21882a-176c-40b1-a120-c36b70e63d83.png)

   * Private routing table이다. 기존에 있었던 0.0.0.0/0을 허용한 라우팅을 제거하고 새로운 라우팅이 추가되었다.
       대상 : **pl-78a54011**
     은 엔드포인트가 생성되면서 2~3분 뒤에 private routing table에 자동적으로 추가되는 라우팅이다.
     
       ![image](https://user-images.githubusercontent.com/33750210/137658625-b4abbb88-2515-42eb-ae28-e8e3895f4a2d.png)

   * 접두사 목록 이름을 확인해보면 끝 부분에 **s3**가 있는 것을 알 수 있다. 이 라우팅의 의미는 private subnet에서 s3로 가려는 패킷은 s3로 보낸다는 뜻이다.



## Private instance에서 s3 접근하기

```shell
$ sudo apt update
$ sudo apt install awscli
```

위 명령어로 **awscli**를  설치한다.
```shell
$ aws s3 ls --region ap-northeast-2
```

s3에 있는 버킷 목록을 보여준다.

![image](https://user-images.githubusercontent.com/33750210/137658738-10b01783-96fc-4fb5-8641-a80186aac704.png)

  "insomenia-vpc-endpoint-check"는 내가 실습을 위해 s3에 생성한 버킷의 이름이다.

* **실습이 종료되면 ec2를 종료한다.**
* **VPC도 종료를 해준다. VPC는 요금이 나가지는 않지만 갯수 제한이 있기 때문이다.**



## 정리

   NAT Gateway로 충분히 외부 네트워크의 서비스를 사용할 수 있지만 보안상 네트워크가 노출되어서 해커에게 공격받을 수 있다. 그래서 private subnet의 instance가 원하는 서비스만 이용하도록 해당 네트워크만 열어줘야 한다. 이 때 사용하는 것이 VPC endpoint 서비스다.  이번 실습에서 s3 서비스를 이용하기 위해서 private instance에게 IAM(s3_fullaccess) 권한을 줬다. 그리고 private route table과 연결되는 s3권한을 가진 endpoint를 생성했다. VPC endpoint는 생성이 되면 자동적으로 endpoint와 연결이 된 라우팅 테이블에 추가된다.
   Private route table에서 대상이 0.0.0.0/0인 row를 지웠지만 private instance는 s3 서비스를 이용할 수 있었다. 반면에 다른 서비스는 이용이 불가능하다. 이렇게 private instance가 이용하는 서비스만 제한적으로 열어주는 것이 보안상 좋다.