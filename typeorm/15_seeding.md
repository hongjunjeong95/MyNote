# Seeding

## Installation

```sh
yarn add typeorm-seeding
yarn add -D @types/faker
```

## package.json

```js
"scripts":{
  ...
  "seed:config": "ts-node ./node_modules/typeorm-seeding/dist/cli.js config",
  "seed:run": "ts-node ./node_modules/typeorm-seeding/dist/cli.js --configName ormconfig.ts seed"
}
```

## ormconfig.ts

```typescript
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

import {
  User
} from './src/database/entities/index';

const config: TypeOrmModuleOptions = {
  type: 'postgres',
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_DATABASE,
  schema: process.env.DB_SCHEMA,
  synchronize: false,
  keepConnectionAlive: true,
  logging: true,
  entities: [
    User
  ],
  // entities: ['./**/*.entity.js'],
  migrations: ['dist/database/migrations/**/*.js'],
  cli: {
    entitiesDir: 'src/**/entities',
    migrationsDir: 'src/database/migrations'
  },
  // @ts-ignore
  seeds: ['src/database/seeds/**/*.seed.ts'],
  factories: ['src/database/factories/**/*.factory.ts'],
  baseUrl: './'
};

export = config;

```

seeding을 이용할 때 주의할 점

1. @같은 absolute path가 안 된다.
2. glob 문법이 안 되므로 모든 entity를 import 해야 한다.
3. docker에서 실행한다면 docker shell에 접속해서 명령어를 실행하라
4. `yarn run seed:run -s <seed class name>`

## user.seed.ts

```typescript
import { Factory, Seeder } from 'typeorm-seeding';

import { User } from '../entities/users/user.entity';

export class CreateUsers implements Seeder {
  public async run(factory: Factory): Promise<void> {
    await factory(User)().createMany(50);
  }
}
```

## user.factory.ts

```typescript
import Faker from 'faker';
import { define } from 'typeorm-seeding';

import { User } from '../entities/users/user.entity';
import {
  Gender,
} from '../entities/enums/index';

const userFaker = (faker: typeof Faker) => {
  const user = new User();

  user.userSocialId = faker.internet.password();
  user.nickname = faker.internet.userName();
  user.snsEmail = faker.internet.email();
  user.imageUrl = `https:source.unsplash.com/6${faker.random.number({
    min: 10,
    max: 99
  })}x603`;
  user.point = faker.random.number({
    min: 0,
    max: 50000
  });
  user.loginTime = faker.date.past(1);
  user.memo = faker.lorem.sentence(3);
  user.pushKey = faker.internet.password();
  user.birthYear = faker.random.number({
    min: 1960,
    max: new Date().getFullYear()
  });
  user.gender = faker.random.objectElement<Gender>(Gender);

  return user;
};

define(User, userFaker);
```

##### References

* [typeorm-seeding](https://www.npmjs.com/package/typeorm-seeding#-installation)
* [faker](https://github.com/faker-js/faker)