# Flyway 마이그레이션

## **기존 테이블 정보, 데이터만 있는 경우**

기존 테이블 정보, 데이터만 있고, flyway_schema_history 테이블은 없는 경우에는 다음처럼 작업하면 된다. 

1.  application 설정 파일에 flyway.baselineOnMigrate 를 true로 추가한다. 원래는 'flyway_schema_history'를 검색하고 없다면 직접 생성하는데, 스프링 부트 2 이상의 경우 아래 해당 옵션을 줘야 생성이 되었다. 

```
spring.flyway.baselineOnMigrate = true 
```

2. spring.flyway.locations 위치에 V1에 해당하는 migration script를 작성한다. 이때 반드시 스크립트 파일이 있어야 한다. 특별한 버전1이 없이, 그냥 기존 테이블과 데이터를 유지하고자 해도 빈 script를 작성해야 한다. 

```
# 빈 파일이어도, V1에 해당하는 파일 자체는 있어야한다.
```

*pring.flyway.locations의 디폴트 위치는 main/java/resources/db/migration이다.*

3. 끝

## flyway_schema_history 테이블까지 있는 경우

기존 테이블 정보, 데이터만 있고, flyway_schema_history 테이블까지 있는 경우에는 다음처럼 작업하면 된다.

1.  application 설정 파일에 flyway.baselineOnMigrate 를 false로 추가한다. 앞선 flyway history 테이블이 있는 경우와 반대로, baselineOnMigration을 false로 한다.

```
spring.flyway.baselineOnMigrate = false
```

 2-1. migration script가 남아있지 않은 경우, 히스토리 내역 최신보다 더 높은 버전으로 빈 스크립트를 추가하고 실행한다. 예를 들면 히스토리 최신 버전이 4라면, 'V5_empty.sql' 같은 빈 스크립트 파일이라도 작성하고 실행해야 한다. 이때 반드시 스크립트 파일이 있어야 한다. 

```
# 빈 파일이어도, 파일 자체는 있어야한다.
```

 2-2. migration script가 남아있는 경우 spring.flyway.locations 위치에 기존 히스토리 내역과 일치하는 스크립트를 추가하고 실행한다.

3. flyway_schema_history 테이블에서 <<Flyway Base>> 이 표시됨을 확인할 수 있을 것이다.

![img](https://blog.kakaocdn.net/dn/VkDS2/btraDz0uvoI/suzHAifo8GF82NRIyJR5Ik/img.png)