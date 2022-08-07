# Setting

> [TypeORM](https://github.com/typeorm/typeorm) is definitely the most mature Object Relational Mapper (ORM) available in the node.js world. Since it's written in TypeScript, it works pretty well with the Nest framework.



## Installation

```shell
$ yarn add @nestjs/typeorm pg psql typeorm
```



## Module Setting

### database 모듈 생성

```shell
$ nest g mo database
```



### databse.module.ts

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigService } from '@nestjs/config';

@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => config.get('database'),
    }),
  ],
})
export class DatabaseModule {}
```

ConfigService와 useFactory를 이용해서 바로 databse config 설정을 할 수 있다. inject를 이용해서 ConfigService를 주입하고 useFactory를 이용해서 databse config 값을 가져온다.



## Database Config 설정



### config.module.ts

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule as NestConfigModule } from '@nestjs/config';

import DatabaseConfig from './database.config';

@Module({
  imports: [
    NestConfigModule.forRoot({
      isGlobal: true,
      envFilePath: process.env.NODE_ENV === 'dev' ? '.env.dev' : '.env.test',
      ignoreEnvFile: process.env.NODE_ENV === 'production',

      load: [
        () => ({ database: DatabaseConfig() }),
      ],
    }),
  ],
})
export class ConfigModule {}
```



### database.config.ts

```typescript
import { TypeOrmModuleOptions } from '@nestjs/typeorm';
import { join } from 'path';

export default () =>
  ({
    type: process.env.DB_DIALECT || 'postgres',
    ...(process.env.DATABASE_URL
      ? { url: process.env.DATABASE_URL }
      : {
          host: process.env.DB_HOST || 'localhost',
          port: parseInt(process.env.DB_PORT, 10) || 5432,
          username: process.env.DB_USERNAME || 'hongjun',
          password: process.env.DB_PASSWORD || '12345',
          database: process.env.DB_DATABASE || 'houpang-v1',
        }),
    logging: process.env.NODE_ENV !== 'prod' && process.env.NODE_ENV !== 'test',
    entities: ['dist/**/*.entity.js'],
    migrations: [join(__dirname, '../', 'database/migrations/**/*.ts')],
    synchronize: true,
    dropSchema: process.env.DB_DROP_SCHEMA === 'true',
    migrationsRun: process.env.DB_MIGRATIONS_RUN === 'true',
  } as TypeOrmModuleOptions);

```

  DB server를 외부에 두면 URL을 받을 수 있다. 그러면 URL 값 하나에 DB server와 db 정보가 다 있으므로 URL을 사용하고 그렇지 않다면 host, port, username, password, database(db 이름) 등을 설정한다.



<table>
    <thead>
        <th>항목</th>
        <th>의미</th>
    </thead>
    <tbody>
        <tr>
            <td style="width:200px;">logging</td>
            <td>DB log를 출력</td>
        </tr>
        <tr>
            <td>entities</td>
            <td>
                entities는 Entity들의 파일 위치를 정의한다. Entity는 테이블 스키마를 정의하는 곳이다. typeorm은 엔티티를 확인해서 마이그레이션 파일을 생성하고 DB에 적용한다.
            </td>
        </tr>
        <tr>
            <td>migrations</td>
            <td>마이그레이션 파일 위치를 정의한다.</td>
        </tr>
        <tr>
            <td>synchronize</td>
            <td>true로 설정되면 마이그레이션 파일을 따로 generate 후 run을 하지 않아도 entity 파일을 저장하기만 해도 바로 DB에 적용되어 테이블이 생성된다. 이 설정 값은 개발할 때 편리하지만 프로덕션 단계에서는 절대로 하면 안 된다. 또한, 개발을 할 때도 최대한 지양하고 <strong>마이그레이션을 통해서 DB를 수정할 것을 권장한다.</strong></td>
        </tr>
        <tr>
            <td>dropSchema</td>
            <td>DB 연결이 성공할 때마다 스키마를 drop한다. 이 옵션은 절대 production에서 사용하면 안 된다. 그렇지 않으면 모든 production 데이터를 잃을 수 있다. 이 옵션은 debug나 development 단계에서 유용하다.</td>
        </tr>
        <tr>
            <td>migrationsRun</td>
            <td>앱이 시작할 때마다 자동으로 마이그레이션을 run할 것인지에 대한 옵션이다.</td>
        </tr>
    </tbody>
</table>
