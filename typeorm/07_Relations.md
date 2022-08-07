# Relations 1:1, 1:M, M:N



## One-to-One

### uni-directional

```typescript
// profile.entity.ts

import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class Profile {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    gender: string;

    @Column()
    photo: string;

}
```

```typescript
// user.entity.ts

import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from "typeorm";
import { Profile } from "./Profile";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToOne(() => Profile)
    @JoinColumn()
    profile: Profile;

}
```

  사용자(User)와 사용자의 찜 목록(Profile)이 서로 1:1 관계를 맺는다. 이 때 Profile이 User에게 종속되므로 User에서 Profile을 참조하도록 User에서 @OneToOne과 @JoinColumn 데코레이터를 추가한다. @OneToOne을 사용할 때는 반드시 @JoinColumn을 추가해줘야 한다. 그 후 마이그레이션을 하면 User 테이블에 @JoinColumn에 의해 createdBy**Id** 칼럼이 생성될 것이다. 반면에 Profile entity에서는 @OneToOne 데코레이터를 추가하지 않는다.

### bi-directional

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Profile {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    gender: string;

    @Column()
    photo: string;

    @OneToOne(() => User, user => user.profile) // specify inverse side as a second parameter
    user: User;

}
```

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from "typeorm";
import { Profile } from "./Profile";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToOne(() => Profile, profile => profile.user) // specify inverse side as a second parameter
    @JoinColumn()
    profile: Profile;

}
```

**uni-directional**은 한 반향에서만 테이블을 참조할 수 있었다. **bi-directional**은 양방향에서 서로의 관계를 참조할 수 있다. 이 때는 양 Entity 모두 @OneToOne 데코레이터를 추가하고 @JoinColumn()은 User Entity에만 추가해준다.

```typescript
// profile이 user를 join할 수 있게 됨

const profiles = await profileRepository.findOne({
  relations : ["user"]
});
```

**uni-directional**에서는 user를 join해서 가져올 수 없었는데, Profile Entity에 @OneToOne 컬럼을 추가해줘서 이제는 가능하다.

## Many-to-One & One-to-Many

```typescript
// user.entity.ts

import {
  IsEmail,
  IsString,
  MinLength,
} from 'class-validator';
import { Column, Entity, OneToMany } from 'typeorm';

import { CommonEntity } from '@apis/common/entities/common.entity';

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
  
  @OneToMany((type) => Product, (product) => product.provider)
  products: Product[];
}
```

```typescript
// product.entity.ts

import { IsNumber, IsString, Max, Min } from 'class-validator';
import {
  Column,
  Entity,
  ManyToOne,
  RelationId,
} from 'typeorm';

import { CommonEntity } from '@apis/common/entities/common.entity';
import { User } from '@apis/users/entities/user.entity';

@Entity()
export class Product extends CommonEntity {
  @Column()
  @IsString()
  name: string;

  @ManyToOne((type) => User, (user) => user.products, {
    onDelete: 'CASCADE',
  })
  provider: User;

  @RelationId((product: Product) => product.provider)
  providerId: number;

  @Column()
  @IsNumber()
  @Min(0)
  price: number;
}

```

  1 명의 유저가 여러 개의 상품과 관계를 맺고 있다. 다수의 products가 하나의 user를 가리키고 있다. 

```typescript
@OneToMany((type) => Product, (product) => product.provider)
products: Product[];
```

  이 때 User Entity에서는 @OneToMany를 사용해서 Product를 가리킨다. 첫번째 인자로 Product를 반환하는 익명 함수를 정의하고 두 번째 인자로 product가 가리키고 있는 User의 column name(product.provider)을 반환하는 익명함수를 넘긴다. 

```typescript
@ManyToOne((type) => User, (user) => user.products, {
  onDelete: 'CASCADE',
})
provider: User;
```

  반면에 Many 쪽을 담당하는 Product Entity에서는 @ManyToOne을 사용하고 첫 번째 인자로는 One에 해당하는 User를 반환하는 익명함수를 넘겨주고 두 번째 인자로는 user측에서 products를 가리키고 있는 user.products를 반환한다. 세 번째 인자로는 컬럼의 옵션을 설정할 수 있는데, product를 생성한 User가 삭제되면 product도 같이 다 삭제되도록 onDelete: 'CASCADE'를 설정할 수 있다. 



### @RelationId

**@RelationId **적용 전

```typescript
const product = await this.products.findOne({
        where: {
          provider:{
          	id: providerId
        	},
        }
      });
```

**@RelationId **적용 후

```typescript
const product = await this.products.findOne({
        where: {
          providerId: providerId
        }
      });
```

@RelationId를 추가하면 객체 안으로 들어가서 where 옵션을 주지 않아도 바로 providerId 값을 기준으로 공급자가 providerId인 상품들만 가져올 수 있다.



## Many-to-Many

```typescript
// product.entity.ts

import { IsNumber, IsString, Max, Min } from 'class-validator';
import {
  Column,
  Entity,
  ManyToOne,
  RelationId,
} from 'typeorm';

import { CommonEntity } from '@apis/common/entities/common.entity';
import { User } from '@apis/users/entities/user.entity';

@Entity()
export class Product extends CommonEntity {
  @Column()
  @IsString()
  name: string;

  @ManyToOne((type) => User, (user) => user.products, {
    onDelete: 'CASCADE',
  })
  provider: User;

  @RelationId((product: Product) => product.provider)
  providerId: number;

  @Column()
  @IsNumber()
  @Min(0)
  price: number;
}
```

```typescript
// like.entity.ts

import { Entity, JoinColumn, JoinTable, OneToOne } from 'typeorm';

import { CommonEntity } from '@apis/common/entities/common.entity';
import { User } from '@apis/users/entities/user.entity';

@Entity()
export class Like extends CommonEntity {
  @OneToOne(() => User, {
    onDelete: 'CASCADE',
  })
  @JoinColumn()
  createdBy: User;
  
  @ManyToMany(() => Product)
  @JoinTable()
  products: Product[];
}
```

  상품(Product)과 사용자의 찜 목록(Like)가 서로 M:N 관계를 맺는다. 하나의 상품은 여러 개의 찜 목록을 가리킬 수 있고 동시에 하나의 찜 목록도 여러 개의 상품들을 가리킬 수 있다. @OneToOne과 마찬가지로 @ManyToMany는 한 개의 Entity에서만 사용할 수 있다. 또한, @OneToOne은 @JoinColumn()을 반드시 추가해야 했던 것처럼, @ManyToMany는 @JoinTable()을 반드시 추가해줘야 한다. @JoinTable은 likeId와 productId 두 개의 컬럼을 가졌다.

  @ManyToMany()는 Product나 Like 중 아무데서나 선언해도 상관 없다.