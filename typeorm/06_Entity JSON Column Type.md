# Entity JSON Column Type

Postgres에서 json 타입을 컬럼으로 주고 싶다면 밑에 처럼 하면 된다.

## user.entity.ts

```typescript
import { Column, Entity } from 'typeorm';

import { CommonEntity } from '@common/entities/common.entity';

export class InfoItem {
  @IsNumber()
  id: number;

  @IsString()
  key: string;

  @IsString()
  value: string;
}

@Entity()
export class Item extends CommonEntity {
  @Column({ nullable: true, type: 'json' })
  infos?: InfoItem[];
}
```


단순히 column type을 'json'으로 주고 typescript의 type은 class(InfoItem)을 생성해서 명시해주면 된다.