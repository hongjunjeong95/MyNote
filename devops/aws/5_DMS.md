# AWS DMS(Data Migration Service)

## 시작 전

- 목표 : 이기종 DB서버를 Migration하기
- [여기를 눌러](https://cloudest.tistory.com/20) 기본 VPC & EC2 환경을 구성할 수 있다.
- 이 포스팅은 DMS에 대한 가장 기초적인 On-premise to RDS 마이그레이션을 위한 최소환경으로 구성하였다.

## DMS란?

- DMS란? : 각종 데이터베이스를 DMS를 사용하여 AWS Cloud로 마이그레이션하거나, 온-프레미스 인스턴스 간에(AWS 클라우드 설정을 통해) 또는 클라우드와 온-프레미스 설정의 조합 간에 마이그레이션할 수 있다.
- DMS는 Free-Tier를 제공하지 않는 서비스이기 때문에 약간의 과금이 필요하다. (실습만 진행하고 삭제하면 1$ 이내로 가능하다)

## Srouce DB

- MySQL engine의 RDS
- DB서버는 보통 Private공간에서 운영되는데 DMS는 IP를 기반으로 Origin DB에 접근하기 때문에 DMS를 위해 잠시 공인 IP를 설정해준 것이다.
- DB서버에 공인 IP를 설정하지 않고 DB를 Migration하려면 AWS Direct Connect 서비스를 사용하거나 VPN을 구축해 진행하는 방법도 있다.

## Target DB

- PostgreSQL engine의 RDS

## Replication instances

1. Create replication instance

   ![image](https://user-images.githubusercontent.com/96710287/148012429-ce25afb5-9057-462b-bc31-a2e592c01de9.png)

2. Replication instance configuration-1

   <img src="https://user-images.githubusercontent.com/96710287/148012761-a52e3b2f-02b7-420d-a445-08851aa76518.png" alt="image" style="zoom:50%;" />

   * 메모리 부족으로 에러가 날 수 있어서 메모리는 최소 8GB이상으로 instance class를 선택한다.

3. Replication instance configuration-2

   <img src="https://user-images.githubusercontent.com/96710287/148012816-e74d0f26-8719-4465-95e9-fa2248cb3142.png" alt="image" style="zoom:50%;" />

   - Publicly accessible : DMS가 퍼블릭 IP를 이용해서 연결접근을 시도하는데 추천하지 않는다. 보안 이슈와 성능이 많이 느려진다.

4. Network configuration

   <img src="https://user-images.githubusercontent.com/96710287/148013089-5b27dd42-065a-4ab2-93c1-4fc0490d01bb.png" alt="image" style="zoom:50%;" />

5. Maintenance

   <img src="https://user-images.githubusercontent.com/96710287/148013262-108fb1fe-72c9-47b9-99eb-08e7a25cc4f3.png" alt="image" style="zoom:50%;" />

   - Minor version automatic upgrade : No
     - 유지보수 작업이 걸릴 때 언제 수행할지 설정한다. 마이너 버전 자동 업그레이드 항목은 가능하면 '아니요'로 선택한다. 자동으로 하게 되면 AWS에서 유지보수 공지를 한 날짜에 인위적으로 업그레이드를 진행하게 되며 이 때 시스템 다운이 일어나게 된다.

6. Create

   <img src="https://user-images.githubusercontent.com/96710287/148013426-dd4eee97-711e-4025-a733-a100b87efe41.png" alt="image" style="zoom:50%;" />

## Source Endpoint

1. Create endpoint

   ![image](https://user-images.githubusercontent.com/96710287/148013502-c97ba5a8-483e-4032-92b0-b19e65868079.png)

2. Select **Source endpoint**

   <img src="https://user-images.githubusercontent.com/96710287/148013817-67b10baa-69d4-46a9-a458-75cdc474c63a.png" alt="image" style="zoom:50%;" />

   - RDS Instance를 선택하면 AWS가 password를 제외하고 자동으로 input에 값을 채워준다.

3. Endpoint Configuration

   <img src="https://user-images.githubusercontent.com/96710287/148014043-b2ea768a-7dc7-437f-b946-ea1459baf412.png" alt="image" style="zoom:45%;" />

4. test

   <img src="https://user-images.githubusercontent.com/96710287/148014287-cd9fbcf5-7b7a-46af-822a-c9a13f606b20.png" alt="image" style="zoom:50%;" />

   1. Run test로 statusrk **successful**이 뜨는 것을 확인
      * **Create endpoint**를 하지 않더라도 test를 했다면 endpoint는 생성된다.
   2. Ceate endpoint

## Target Endpoint

Source endpoint와 똑같은 과정을 수행한다. 다만 Endpoint type을 Target endpoint로 선택한다.

<img src="https://user-images.githubusercontent.com/96710287/148015296-d1bf32a9-ad8c-40d0-a188-1b57fac768c3.png" alt="image" style="zoom:50%;" />

## DB Migration tasks

1. Create db migration task

   ![image](https://user-images.githubusercontent.com/96710287/148016320-71799fa5-3942-498f-bcf3-1794e02bc71c.png)

2. Task configuration

   <img src="https://user-images.githubusercontent.com/96710287/148016431-73787e28-1ec1-4cc6-9a5e-b4e365ce2cf4.png" alt="image" style="zoom:50%;" />

   - Migration type
     - Migrate existing data : 소스쪽의 모든 테이블과 데이터에 대해서만 복제
     - Migrate existing data and replicate ongoing changes : 소스쪽의 모든 테이블과 데이터를 복제하고 지속적으로 변경되는 소스와 sync를 맞춘다.
       - 실시간으로 sync가 된다고 하지만 어느 정도 지연시간이 있다.
     - Replicate data changes only : 현재 시점부터 CDC 모드만 시작한다. 이전의 데이터는 복제하지 않고 추후에 changes 만 복제한다.

3. Task settings

   <img src="https://user-images.githubusercontent.com/96710287/148016902-c593c60b-312a-48b1-9440-130b574e4843.png" alt="image" style="zoom:50%;" />

   - Target table preparation mode : Drop tables on target
   - Include LOB columns in replication : Don't include LOB columns

   <img src="https://user-images.githubusercontent.com/96710287/148016913-478124ba-4237-41b2-8ef2-cf618d5f7ecf.png" alt="image" style="zoom:50%;" />

4. Table mappings

   <img src="https://user-images.githubusercontent.com/96710287/148017303-35357061-bfa6-4565-973d-b2fbc7e9866c.png" alt="image" style="zoom:40%;" />

   Selection rules에 어떤 스키마와 테이블을 마이그레이션 할 것인지 규칙을 추가한다.

5. Etc...

   <img src="https://user-images.githubusercontent.com/96710287/148017146-0434c6e4-06e9-474e-9686-5658cd7a2cbe.png" alt="image" style="zoom:40%;" />