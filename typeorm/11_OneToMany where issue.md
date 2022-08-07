# OneToMany where issue



## Issue

  Category(One)와 Products(Many)의 관계가 있다고 하자. 관계형 DB에서는 Many가 One을 가리킨다. 그래서 Many는 One의 id 값을 가지고 있는 반면에 One은 Many Id 값을 가지고 있지 않다. 이 뜻은 무엇이냐? typeorm에서 find의 where option을 줄 때 products(Many)는 categoryId를 기준으로 products를 가져올 수 있지만 category(One)는 productId를 기준으로 category 값을 얻어낼 수 없다. 왜냐하면 category는 productId 값을 가지고 있지 않기 때문이다.

  밑에 코드를 보면서 에러를 살펴보고 이를 어떻게 해결할 수 있는지 살펴보자.



## Example code for issue

### category.entity.ts

```typescript
import { IsString, Length } from 'class-validator';
import { Column, Entity, OneToMany } from 'typeorm';

import { Core } from '@apis/common/entities/common.entity';
import { Product } from '@apis/products/entities/product.entity';

@Entity()
export class Category extends Core {
  @Column({ unique: true })
  @IsString()
  @Length(5)
  name: string;

  @OneToMany((type) => Product, (product) => product.category)
  products: Product[];
}
```

  Category(One)는 procuts(Many)를 참조하고 있다.



### product.entity.ts

```typescript
import { IsString } from 'class-validator';
import { Column, Entity, ManyToOne, RelationId } from 'typeorm';

import { Category } from '@apis/categories/entities/category.entity';
import { Core } from '@apis/common/entities/common.entity';

@Entity()
export class Product extends Core {
  @Column()
  @IsString()
  name: string;

  @ManyToOne((type) => Category, (category) => category.products, {
    onDelete: 'SET NULL',
  })
  category: Category;

  @RelationId((product: Product) => product.category)
  categoryId: number;
}
```

  product(Many)는 category(One)를 참조하고 있다. 동시에 categoryId 값을 가지고 있다.



### categories.service.ts

```typescript
import { Injectable } from '@nestjs/common';
import { Repository } from 'typeorm';

import {
  FindByCategoryNameInput,
  FindByCategoryNameOutput,
} from '@apis/categories/dtos/find-category-by-name.dto';

@Injectable()
export class CategoriesService {
  constructor(
    private readonly categories: CategoryRepository,
  ) {}
  
   async findCategoryByProductId({ productId }: { productId: number }) {
    const category = await this.categories.findOne({
      where:{
        products:[
          {
            id: productId
          }
        ]
      },
    });
    return category;
  }
}
```

```shell
"message": "Cannot query across one-to-many for property products",
```

`findCategoryByProductId`에서  products의 id를 기준으로 category를 찾으려고 하면 위 error message가 출력된다. typeorm의 repository 방식으로는 many field를 기준으로 one field를 찾을 수 없는 것이다. 이것을 그럼 어떻게 해결해야 할까? 어쩔 수 없이 QueryBuilder(이하 qb)를 이용해야 한다. 순순 qb를 이용해도 되지만 여기서는 repository 방식을 이용하되 qb를 결합해서 사용할 것이다. 총 두 가지 방식이 있다.



## Example code for solution - 1

```typescript
import { Injectable } from '@nestjs/common';
import { Repository } from 'typeorm';

import {
  FindByCategoryNameInput,
  FindByCategoryNameOutput,
} from '@apis/categories/dtos/find-category-by-name.dto';

@Injectable()
export class CategoriesService {
  constructor(
    private readonly categories: CategoryRepository,
  ) {}
  
   async findCategoryByProductId({ productId }: { productId: number }) {
     const category = await this.categories
       .createQueryBuilder('category')
       .leftJoinAndSelect('category.products', 'product')
       .where('product.id = :id', { id: productId })
       .getOne();
    return category;
  }
}
```

`await this.categories`까지는 repository 코드이고 `createQueryBuilder`부터는 qb 코드다.

* createQueryBuilder('category') : 여기서부터 qb 코드를 작성하겠다는 것을 알린다. 그리고 앞에 `this.categories`가 있다고 하더라고 어떤 테이블을 사용할 것인지 alias(category)를 줘야 한다.
* leftJoinAndSelect('category.products', 'product') : product 값을 참조하기 위해서 category.products를 조인한다.
* where('product.id = :id', { id: productId }) : product.id 값을 기준 값으로 한다.
* getOne(); : category 한 개를 찾는다.



## Example code for solution - 2

```typescript
import { Injectable } from '@nestjs/common';
import { Repository } from 'typeorm';

import {
  FindByCategoryNameInput,
  FindByCategoryNameOutput,
} from '@apis/categories/dtos/find-category-by-name.dto';

@Injectable()
export class CategoriesService {
  constructor(
    private readonly categories: CategoryRepository,
  ) {}
  
   async findCategoryByProductId({ productId }: { productId: number }) {
    const category = await this.categories.findOne({
      join: {
        alias: 'category',
        innerJoin: { products: 'category.products' },
      },
      where: (qb) => {
        qb.where('products.id = :id', { id: productId });
      },
    });
    return category;
  }
}
```

좀 더 repository 방식과 가깝다. where에서만 qb를 사용하여 product에 접근했다.

총 두 가지 방식으로 Many field 값을 기준으로 One에 접근할 수 있다.



##### References

* QueryBuilder
* [Typeorm VS Prisma](https://www.prisma.io/docs/concepts/more/comparisons/prisma-and-typeorm#relation-filters)
