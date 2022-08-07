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