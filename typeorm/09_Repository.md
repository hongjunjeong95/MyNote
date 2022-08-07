# Working with Repository



TypeORM에서 ORM을 사용하는 방법은 다양하다. 그 중 Repository 방식을 사용할 것이다.



## user.entity.ts

```typescript
import { InternalServerErrorException } from '@nestjs/common';
import {
  IsEmail,
  IsEnum,
  IsString,
  MinLength,
} from 'class-validator';
import { BeforeInsert, BeforeUpdate, Column, Entity, OneToMany } from 'typeorm';
import * as bcrypt from 'bcrypt';

import { CommonEntity } from '@apis/common/entities/common.entity';
import { Product } from '@apis/products/entities/product.entity';

export enum UserRole {
  Consumer = 'Consumer',
  Provider = 'Provider',
  Admin = 'Admin',
}

registerEnumType(UserRole, { name: 'UserRole' });

@Entity()
export class User extends CommonEntity {
  @Column({ unique: true, type: 'text' })
  @IsEmail()
  email: string;

  @Column({ unique: true })
  @IsString()
  username: string;

  @Column()
  @MinLength(8)
  password: string;

  @Column({ type: 'enum', enum: UserRole, default: UserRole.Consumer })
  @IsEnum(UserRole)
  role: UserRole;

  @OneToMany((type) => Product, (product) => product.provider)
  products: Product[];
}
```

  user 스키마를 정의하는 user entity를 생성한다. 그 후 마이그레이션을 생성한 후 적용.



## users.module.ts

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

import { User } from '@apis/users/entities/user.entity';
import { UsersService } from '@apis/users/users.service';
import { UsersController } from '@apis/users/users.controller';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

  **users.serivce.ts**에서 사용해야 하므로 User Entity를 DI 해줘야 한다. imports에 `TypeOrmModule.forFeature([User])`로 추가. NestJS는 외부 모듈을 사용해야 할 때 무조건 DI를 해줘야 한다(Global 제외). **UsersService**에서 추가로 더 많은 Entity를 필요로 한다면 `.forFeature([])`에 추가해주면 된다.



## users.service.ts

```typescript
import { Response } from 'express';
import { HttpException, HttpStatus, Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

import {
  CreateAccountInput,
  CreateAccountOutput,
} from '@apis/users/dtos/create-account.dto';
import { User } from '@apis/users/entities/user.entity';

@Injectable()
export class UsersService {
  constructor(
  // consturctor에 @InjectRepository({Entity})를 추가하고
  // 하단에 private readonly {repository name} : Repository<Entity> 추가 해줘야지만 Repository ORM을 사용 가능
    @InjectRepository(User)
    private readonly users: Repository<User>,
  ) {}

  async createAccount({
    email,
    username,
    password,
  }: CreateAccountInput): Promise<CreateAccountOutput> {
    try {
      const existedEmail = await this.users.findOne({ email });
      if (existedEmail) {
        return { ok: false, error: '계정이 있는 이메일입니다.' };
      }

      // this.users.create를 하면 메모리 상에서 데이터가 생성되는 것이고
      // create가 반환하는 값을 this.users.save로 저장해야지만 DB에 저장된다!!
      // update를 사용해서 수정할 때도 마찬가지로 save를 이용해서 저장해야 한다.
      const user = await this.users.save(
        this.users.create({
          email,
          username,
          password,
        }),
      );

      return { ok: true };
    } catch (e) {
      return { ok: false, error: '계정을 생성할 수 없습니다.' };
    }
  }
}

```

* consturctor()에 위 주석처럼 @InjectRepository를 해줘야 Repository ORM을 사용할 수 있다.
* findOne은 하나의 row를 찾는 orm이다. {} 옵션 값으로 column 값을 주면 해당 칼럼의 value값을 가진 row(instance)를 찾아준다. 만약 findOne(1);을 사용한다면 기본적으로 id=1인 user를 찾아준다. **하지만 이 방법은 권장하지 않는다. 만약 user를 찾지 못 했을 경우 예외 처리가 되어야 하는데, findOne은 id 값이 가장 첫 번째인 user를 반환하기 때문이다.**



<span style="font-size:2rem;font-style:italic;">**더 많은 Repository api와 find option은 문서를 참조**</span>

* [TypeORM Repository APIs](https://typeorm.io/#/repository-api)
* [TypeORM find options](https://typeorm.io/#/find-options)

