# Entity Array Column Type

Postgres에서 `Array<string>` 타입을 컬럼으로 주고 싶다면 밑에 처럼 하면 된다.

## user.entity.ts

```typescript
import { Column, Entity } from 'typeorm';

import { CommonEntity } from '@common/entities/common.entity';


@Entity()
export class User extends CommonEntity {
  @Column('text', { array: true })
  images: string[];
}
```


@Column의 타입으로 string이 아니라 text를 주고 옵션으로 array를 true로 해준다. 그러면 postgres column에 string 형 배열을 저장할 수 있다.