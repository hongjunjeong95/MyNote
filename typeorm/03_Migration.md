# Migration



## package.json

```json
"scripts": {
    "typeorm": "cross-env NODE_ENV=dev node --require ts-node/register ./node_modules/typeorm/cli.js",
    "migration:generate": "yarn run typeorm migration:generate",
    "migration:run": "cross-env NODE_ENV=dev ts-node --transpile-only ./node_modules/typeorm/cli.js migration:run"
},
```

"scripts"에 **typeorm** 명령어를 추가. 특히 `migration:run` 같은 경우에는 .ts 파일이 아닌 .js 파일을 찾으므로 먼저 마이그레이션 파일을 트랜스 컴파일을 한 후 실행시켜줘야 한다. 그래서 **ts-node --transpile-only** 를 추가한 것이다.

  cross-env NODE_ENV=dev는 ConfigModule의 envFilePath를 설정하기 위한 명령어다. NODE_ENV의 값에 따라 .env이나 .env.development가 될 수 있다. ormconfig.js에서 process.env를 사용하기 때문에 ConfigModule의 envFilePath 설정을 잘 확인해야 한다.



## Entity

```typescript
import { Column, Entity } from 'typeorm';

import { CoreEntity } from '../../common/entities/common.entity';

@Entity()
export class User extends CoreEntity {
  @Column({ unique: true })
  email: string;

  @Column({ unique: true })
  username: string;

  @Column()
  password: string;
}
```

  Entity 파일은 다음 챕터에서 설명한다. 테이블 스키마라고 생각하면 된다.



## Commands

### generate migration file

```shell
$ yarn run migration:generate -n <migration file name>
```

  Entity 파일들을 보고 마이그레이션 파일을 생성한다. -n 옵션을 줘서 마이그레이션 파일 name을 줄 수 있다. `src/database/migrations/TIMESTAMPE-<migration name>` 파일이 생성된다.

![image](https://user-images.githubusercontent.com/33750210/141258586-74493661-bf8d-4af8-af73-b52e72e0dff8.png)

<img src="https://user-images.githubusercontent.com/33750210/141259284-7353963a-dfaf-4fc4-9094-e6653af90bd7.png" alt="image" style="zoom:50%;" />

```typescript
import { MigrationInterface, QueryRunner } from 'typeorm';

export class firstMigration1636683224086 implements MigrationInterface {
  name = 'firstMigration1636683224086';

  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(
      `CREATE TABLE "user" ("id" uuid NOT NULL DEFAULT uuid_generate_v4(), "createdAt" TIMESTAMP NOT NULL DEFAULT now(), "updatedAt" TIMESTAMP NOT NULL DEFAULT now(), "email" character varying NOT NULL, "username" character varying NOT NULL, "password" character varying NOT NULL, CONSTRAINT "UQ_e12875dfb3b1d92d7d7c5377e22" UNIQUE ("email"), CONSTRAINT "UQ_78a916df40e02a9deb1c4b75edb" UNIQUE ("username"), CONSTRAINT "PK_cace4a159ff9f2512dd42373760" PRIMARY KEY ("id"))`,
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE "user"`);
  }
}
```

  자동으로 <migration_name>TIMESTAMPE 클래스가 생성되고 메소드로 up과 down이 생성된다. up은 마이그레이션을 수행할 코드가 작성되고 down은 마지막으로 적용된 마이그레이션을 revert하는 코드를 작성한다. **generate**는 신중히 하자!! 문서 최하단에 **generate** issue가 있으니 확인하자!



### run the migration file

```shell
$ yarn run migration:run # migration 파일을 찾아서 DB 테이블을 생성 및 수정
```

<img src="https://user-images.githubusercontent.com/33750210/141397187-5928d26e-e8ce-42f3-99f4-d370d7cd1229.png" alt="image" style="zoom:50%;" />

postgresql MAC GUI인 postico로 테이블이 정상적으로 생성된 것을 확인



## Migration Generate Issue

  `generate`는 Entity 파일과 DB를 비교해서 업데이트 부분을 알아서 작성한다. 하지만 이 컬럼의 이름, 타입 또는 최대 길이 등을 변경할 때 **generate**를 사용하면 해당 테이블의 컬럼을 **DROP** 한 뒤, 재생성이 이뤄지기에 **데이터가 날아가버린다.** 아래 해당 이슈 확인

> https://github.com/typeorm/typeorm/issues/3357

그래서 불편하더라도 **create** 명령어를 사용할 것을 권장한다.

### create migration file

```shell
$ yarn run migration:create -n <migration name>
```

  generate와 다른 점은 자동적으로 코드를 생성하지  않고 자신이 직접 코드를 짜야 한다.

## Migration Run with docker Issue

마이그레이션을 run할 때 dist folder 안에 있는 migration files를 확인한다. docker의 postgres DB에 마이그레이션을 적용하고 싶다면 먼저 로컬 호스트에서 `yarn run build`를 통해 dist folder 안에 migration file을 추가해줘야 한다. 그 후 도커에서 아래 명령어를 실행한다.

```sh
docker exec -it <nest-app container name> sh
yarn run migration:run
```