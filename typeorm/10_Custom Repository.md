# Custom Repository



  사용자가 카테고리 이름으로 해당 카테고리를 찾을 때 DB에는 slug로 저장되는 경우가 일반적이여서 name을 slug로 파싱해줘야 한다. 예를 들어 'Fasion Design'으로 카테고리 이름이 되어 있다면 slug는 'fasion-design'으로 띄어쓰기를 없앤 상태에서 DB에 저장된다. 하지만 사용자는 'fasion-design'으로 검색하지 않기 때문에 category name이 input 값으로 들어왔을 때 서버에서는 이를 slug로 파싱하고 by slug로 category를 검색해야 한다. 하지만 기본적으로 제공하는 repository api로는 한계가 있기 때문에 custom repository를 생성해야 한다.

  밑에는 Custom Repository를 만드는 방법이다.



## category.repository.ts

```typescript
import { EntityRepository, Repository } from 'typeorm';
import { Category } from '@apis/categories/entities/category.entity';

@EntityRepository(Category)
export class CategoryRepository extends Repository<Category> {
  async findOneBySlugUsingName(name: string) {
    const categoryName = name.trim().toLowerCase();
    const slug = categoryName.replace(/ /g, '-');

    const category = await this.findOne({ slug });

    if (!category) {
      return undefined;
    }
    return category;
  }
}
```

  커스텀 레포지토리를 만들기 위해서 `Repository`를 extends 하는 `@EntityRepository()`를 사용해야 한다.



## categories.module.ts

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

import { Product } from '@apis/products/entities/product.entity';
import { CategoriesService } from '@apis/categories/categories.service';
import { CategoriesController } from '@apis/categories/categories.controller';
import { CategoryRepository } from '@apis/categories/repositories/category.repository';

@Module({
  imports: [TypeOrmModule.forFeature([CategoryRepository, Product])],
  providers: [CategoriesService],
  controllers: [CategoriesController],
})
export class CategoriesModule {}
```

  **categories.module.ts**에서는 Category를 `TypeOrmModule.forFeature([])`에 추가하는 것이 아니라 `CategoryRepository`를 추가한다.



## categories.service.ts

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
  async findByCategorySlug({
    name,
  }: FindByCategoryNameInput): Promise<FindByCategoryNameOutput> {
    const category = await this.categories.findOneBySlugUsingName(name);

    if (!category) {
      throw new HttpException(
        {
          status: HttpStatus.BAD_REQUEST,
          error: 'There is no category by name',
        },
        HttpStatus.BAD_REQUEST,
      );
    }

    return {
      ok: true,
      category
    };
  }
}
```

기본적인 Repository와는 `@InjectRepository()`를 사용하지 않고 생성자에 custom repository를 추가하면 된다.

