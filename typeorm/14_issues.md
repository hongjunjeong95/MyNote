# Issues

## MissingDriverError: Wrong driver: "undefined" given

```js
// ormconfig.js

module.exports = {
  type: 'postgres', // 환경변수로 하지말고 하드코딩할 것
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_DATABASE,
  schema: 'onlineworkshop',
  synchronize: false,
  keepConnectionAlive: true,
  logging: ['warn', 'error'],
  entities: ['./**/*.entity.js'],
  migrations: ['dist/database/migrations/**/*.js'],
  cli: {
    entitiesDir: 'src/**/entities',
    migrationsDir: 'src/database/migrations',
  },
  // @ts-ignore
  seeds: ['src/database/seeds/**/*.ts'],
  factories: ['src/database/factories/**/*.ts'],
  baseUrl: './',
};
```

## Typeorm Entity metadata for <Entity>#<Field> was not found

Nestjs는 Module단위로 의존성 관리를 하며, annotation으로 얻은 metadata를 통해 nestjs는 app structure를 정규화한다.

그러하여 Typeorm의 model class(entity)에서 외부의 dependency class가 필요한 경우 'Module'을 통하여 해당 model class에 제공해야 한다. (ex - OneToMany, ManyToOne)

이러한 전제를 생각하여 의존성 주입이 제대로 되었는지 확인한다.

 **Result**

​	-> Module의 의존성 관리를 확인한다. (TypeOrmModule.forFeature)

forFeature에 Field에 해당하는 entity를 추가한다.