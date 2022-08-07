# ormconfig.js

```javascript
module.exports = {
  type: process.env.DB_DIALECT,
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_DATABASE,
  synchronize: false,
  logging: ["warn", "error"],
  entities: ["dist/**/entities/**/*.entity.js"],
  migrations: ["dist/database/migrations/**/*.js"],
  cli: {
    entitiesDir: "src/**/entities",
    migrationsDir: "src/database/migrations",
  },
  seeds: ["src/database/seeds/**/*.ts"],
  factories: ["src/database/factories/**/*.ts"],
  baseUrl: "./",
};
```

챕터 1에서 Typeorm 설정 값을 database.config.ts에서 관리한 것을 보았다. 하지만 typeorm 설정 값만 따로 보관하고 싶을 경우가 있다. ormconfig.(json, yml, js 등) 파일로 이 설정이 가능하다. TypeOrmModule.forRoot()에 설정이 되어 있어도 typeorm은 ormconfig 설정을 우선시하기 때문에 .forRoot()에 있는 값을 무시해버린다. ormconfig.json 파일은 환경 변수가 모두 노출이 되기 때문에 ormconfig.js 파일을 사용할 것을 권장한다.

서버를 실행하거나 마이그레이션 명령을 실행할 때 환경 변수를 읽지 못 하는 경우가 있는데, 그 때는 ConfigModule이 DB Module보다 먼저 왔는지 확인하고 ConfigModule의 envFilePath가 잘 설정되었는지 확인한다.

entities와 migrations는 ts를 읽을 수 없고 js를 읽어야만 한다. 그렇기 때문에 src/ 가 아닌 dist/ 안에 있는 .js 파일을 경로로 설정해줘야 한다.

## database.module.ts

```typescript
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";

@Module({
  imports: [TypeOrmModule.forRoot()],
})
export class DatabaseModule {}
```

app.module.ts가 database.module.ts를 import 하고 database.module.ts는 단순히 TypeOrmModule.forRoot()를 import에 추가하면 된다.

###### 참조

- [ormconfig](https://typeorm.io/#/using-ormconfig)
