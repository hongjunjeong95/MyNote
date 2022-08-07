# Entity Basic



Entity는 데이터베이스 테이블에 맵핑되는 클래스다. @Entity()를 사용해서 entity를 정의할 수 있다. 



## user.entity.ts

```typescript
import { InternalServerErrorException } from '@nestjs/common';
import { IsEmail, IsString, MinLength } from 'class-validator';
import { BeforeInsert, BeforeUpdate, Column, Entity } from 'typeorm';
import * as bcrypt from 'bcrypt';

import { CommonEntity } from '@common/entities/common.entity';

export enum UserRole {
  Consumer = 'Consumer',
  Provider = 'Provider',
  Admin = 'Admin',
}

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
  @Field((type) => UserRole)
  @IsEnum(UserRole)
  role: UserRole;

  @BeforeInsert()
  @BeforeUpdate()
  async hashPassword(): Promise<void> {
    if (this.password) {
      try {
        this.password = await bcrypt.hash(this.password, 10);
      } catch (e) {
        console.error(e);
        throw new InternalServerErrorException();
      }
    }
  }

  async checkPassowrd(password: string): Promise<boolean> {
    try {
      const ok = await bcrypt.compare(password, this.password);
      return ok;
    } catch (error) {
      console.error(error);
      throw new InternalServerErrorException();
    }
  }
}
```

<table>
    <thead>
        <th>항목</th>
        <th>의미</th>
    </thead>
    <tbody>
        <tr>
            <td style="width:200px;">@Entity</td>
            <td>Entity 파일이라는 것을 명시. @Entity 하단에 있는 클래스 이름으로 테이블 생성</td>
        </tr>
        <tr>
            <td>@Column</td>
            <td>테이블의 컬럼을 생성. 여러 옵션 값을 줌으로써 컬럼을 설정. unique를 줘서 같은 row를 생성하지 못 하게 할 수 있고 type을 지정할 수도 있다. 만약 password처럼 type을 지정하지 않으면 typescript의 문법을 따라가서 암묵적으로 타입을 지정한다.</td>
        </tr>
        <tr>
            <td>class-validator</td>
            <td>
                Entity는 클래스이기 때문에 class-validator(@IsString이나 @IsEmail 등)로 유효성 검사를 할 수 있다.
            </td>
        </tr>
        <tr>
            <td>enum type</td>
            <td>enum type을 컬럼에 추가하고 싶다면 UserRole처럼 type을 'enum'으로 해주고 어떤 enum인지 enum:UserRole처럼 추가한다.</td>
        </tr>
        <tr>
            <td>@BeforeInsert & @BeforeUpdate</td>
            <td>password는 절대 평문으로 DB에 저장되면 안 된다. 그렇기 때문에 데이터가 db에 insert 되기 전에, 데이터를 db에 업데이트 하기 전에 처리하는 함수를 작성할 수 있다.</td>
        </tr>
    </tbody>
</table>

Entity는 무조건 Primary Column을 가져야만 한다. 하지만 user.entity.ts는 Primary column이 없다. 대신 엔티티가 클래스라서 CoreEntity를 상속한다.



## common.entity.ts

```typescript
import {
  CreateDateColumn,
  PrimaryGeneratedColumn,
  UpdateDateColumn,
} from 'typeorm';

export abstract class CommonEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

```

<table>
    <thead>
        <th>항목</th>
        <th>의미</th>
    </thead>
    <tbody>
        <tr>
            <td style="width:200px;">@PrimaryGeneratedColumn</td>
            <td>Primay Column을 생성하는 어노테이션이다. 인자로 increment, rowid, uuid 등을 줄 수 있다.</td>
        </tr>
        <tr>
            <td>@CreateDateColumn</td>
            <td>row가 생성된 시점의 date column이다.</td>
        </tr>
        <tr>
            <td>@UpdateDateColumn</td>
            <td>
                row가 업데이트 된 시점의 date column이다.
            </td>
        </tr>
    </tbody>
</table>
  CommonEntity에서는 @Entity를 추가하지 않았고 class에 abstract를 추가했다. CommonEntity는 다른 모든 Entity들이 상속받기 위해 만들어졌다. 그렇기 때문에 CommonEntity 테이블을 만들 필요가 없고 대신 abstract를 추가해서 추상화만 해준다. CommonEntity 덕분에 중복되는 코드를 줄일 수 있다.

## Column Types for Postgres

<img src="https://user-images.githubusercontent.com/33750210/141401659-68af2755-5c8c-452e-88ce-83260c244ef8.png" alt="image" style="zoom:50%;" />