# Flyway 도입기

## 배경

local - dev - prod의 통일성 있는 테이블 버전 관리를 위해 도입.

걱정하지 않고 디비를 마음대로 조작하기 위해선 공용으로 쓰는 데브가 아닌 로컬에 디비를 띄우는게 가장 좋다. 다만 여러 개발자가 동시다발적으로 테이블을 생성하게 되면 테이블 불일치가 발생하게 되는데 flyway로 스키마 버전 관리를 하게 되면 `git pull` 을 하고 migrate 명령어만 치면 손쉽게 로컬 디비에 다른 개발자들이 만든 디비가 생성되거나 `Alter`를 할 수 있다.

그리고 수동으로 디비 컬럼이나 테이블을 조작하게 되면 데브와 가장 중요한 프로드가 불일치하는 경우가 발생하여 500에러가 발생할 수 있는데, 이를 모두 시스템화 시키고 자동화시키면 휴먼에러를 방지할 수 있다.

앞으로 밀리빌리와 에듀빌리 디비의 테이블은 `controlcenter-backend`에서 flyway 중앙관리를 통해 관리한다.

## 설치

### build.gradle.kts

```kotlin
plugins {
    id("org.flywaydb.flyway") version "10.4.1"
}

dependencies {
  runtimeOnly("org.postgresql:postgresql:42.7.1")
  implementation("org.flywaydb:flyway-core:10.4.1")
//    buildscript 쪽에도 추가했지만 여기에도 추가해야 통합 테스트 때 flyway가 작동할 수 있다.
  implementation("org.flywaydb:flyway-database-postgresql:10.4.1")
}
```

### application.yml

```yaml
spring:
	flyway:
    enabled: false
    baseline-on-migrate: true
    baseline-version: 0
    locations: classpath:/db/migration/milyvily
    validate-on-migrate: true
```

- `enabled : true` 면 스프링이 빌드될 때마다 flyway가 자동으로 실행이 되버린다. 그러면 의도치 않게 DDL을 건드리게 되어서 테이블이 조작되어 버린다. 반드시 ****`enabled: false`***를 한다.

- ```
  baseline-on-migrate: true
  ```

   : flyway를 적용하기 전에 이미 테이블들이 있을 때 하는 설정이다. 기존 테이블 정보, 데이터만 있고, flyway_schema_history 테이블은 없는 경우에 설정한다.

  - application 설정 파일에 flyway.baselineOnMigrate 를 true로 추가한다. 원래는 'flyway_schema_history'를 검색하고 없다면 직접 생성하는데, 스프링 부트 2 이상의 경우 아래 해당 옵션을 줘야 생성이 되었다.
  - 반면에 처음 디비 설계를 flyway로 하게 된다면 `baseline-on-migrate: false`로 하면 된다.

- `baseline-version: 0` : baseline이 시작될 때 version이다. 앞으로 flyway로 관리할 `.sql` 파일들은 버전이 1부터 시작하므로 0으로 입력해준다.

- `locations` : flyway가 실행될 때 읽을 DDL 파일들이 위치해있는 경로를 뜻한다.

- `validate-on-migrate` : migration을 하기 전에 유효성 검사를 한다. 즉 validate를 통과하지 못 하면 안전하게 마이그레이션을 실패하게 한다.

## 파일 관리

`main/resources/db/migration/app` 폴더에 .sql 파일들을 추가한다. 이 때 이름 컨벤션이 중요하다. `V{version}__{descrition}.sql` 로 해줘야 한다. 항상 V로 시작하고 그 뒤에 integer 번호가 붙는다. 그리고 무조건 under score(_) 두개를 붙여준 후 뒤에 description을 쓴다.

> 예시) V1__Create_user_table.sql

그리고 flyway를 실행하게 되면 V 뒤에 번호 순서대로 sql 파일을 읽고 db migration을 적용한다. 이 때 잘 적용이 되면 checksum이 적용되는데, 추후에 파일 내용이 바뀌면 체크섬을 통과하지 못 하여 마이그레이션을 진행할 수 없게 막아준다. 그러니 DDL 작성 실수를 이미 적용했다면 파일 수정을 하지 말고 새로운 버전 파일을 만들어서 마이그레이션을 적용하자.

또한, 반복적으로 flyway를 실행해도 체크섬이 같으면 skip을 하기 때문에 이전 파일들이 재실행 될 걱정이 없다.

### 기존에 테이블이 존재한다면 에러가 나서 중지됨

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/44e82b1c-cfcb-4fc9-b21a-53a04dc00787/21d1541d-c746-4b32-81e8-0d1a2497b395/Untitled.png)

## CICD를 위한 flyway CMD

앱 배포 이전에 DB 마이그레이션을 해야 앱에서 DB Error가 발생하지 않는다. CI 단계에서 DB Migration을 진행하는 명령어를 실행해야 한다. gradle 명령어를 task에 등록해서 만들 수 있다.

```kotlin
import org.yaml.snakeyaml.Yaml
import java.io.FileInputStream

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
      // 제거하면 migrateMilyvily 할 때 db 못 찾음
      classpath("org.flywaydb:flyway-database-postgresql:10.4.1")
    }
}

// application.yaml 읽는 함수
fun readYamlConfig(filePath: String): List<Map<String, Any>> {
    val yaml = Yaml()
    FileInputStream(filePath).use { inputStream ->
        val result = mutableListOf<Map<String, Any>>()
        val documents = yaml.loadAll(inputStream)
        for (document in documents) {
            val map = document as? Map<String, Any>
            if (map != null) {
                result.add(map)
            }
        }
        return result
    }
}

val yamlConfig = readYamlConfig("src/main/resources/application.yml").firstOrNull() ?: emptyMap()
val springConfig = yamlConfig["spring"] as? Map<String, Any> ?: emptyMap()
val datasourceConfig = springConfig["datasource"] as? Map<String, Any> ?: emptyMap()
val dbUrl = datasourceConfig["jdbc-url"] as? String ?: "jdbc:postgresql://localhost:5432/your_db"
val dbUser = datasourceConfig["username"] as? String ?: "postgres"
val dbPasswd = datasourceConfig["password"] as? String ?: "12345"

// flyway는 build 안에 있는 migration file을 읽는다.
tasks.register<Copy>("copySqlResources") {
    from("src/main/resources/db/migration")
    into("build/resources/main/db/migration")
}

tasks.register("migrateDb") {
    dependsOn("copySqlResources")
    doLast {
        exec {
            commandLine("./gradlew", "flywayMigrate",
                "-Pflyway.url=${dbUrl}",
                "-Pflyway.user=${dbUser}",
                "-Pflyway.password=${dbPasswd}",
                "-Pflyway.locations=classpath:db/migration/milyvily",
                "-Pflyway.baselineOnMigrate=true",
                "-Pflyway.baselineVersion=0")
        }
    }
}
```

commandLine 옵션을 보면 `application.yml`에서 해준 설정과 똑같다.

```yaml
jobs:
	ci:
		steps:
			- name: Run Flyway Migrations
        run: ./gradlew migrateDb
      # gradle build
      - name: Build with Gradle
        run: ./gradlew build
```

CI 단계에 위 명령어를 `./gradlew build` 전에 넣으면 된다.

## Flyway를 CC에서 분리하는 전략

1. DB Migration만 분리해서 브랜치별로 나눈다.

## References

- https://sabarada.tistory.com/193