# Eager and Lazy Relations



## Eager

Eager relations는 자동적으로 DB에서 entity를 부를 때마다 로드된다.

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from "typeorm";
import { Question } from "./Question";

@Entity()
export class Category {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @ManyToMany(type => Question, question => question.categories)
    questions: Question[];
}
```

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from "typeorm";
import { Category } from "./Category";

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    text: string;

    @ManyToMany(type => Category, category => category.questions, {
        eager: true
    })
    @JoinTable()
    categories: Category[];
}
```

```typescript
const questionRepository = connection.getRepository(Question);

// questions will be loaded with its categories
const questions = await questionRepository.find();
```

  예를 들어서 questions를 로드할 때 eager : true가 아니라면 보통은 id, title, text만 가져온다. 하지만 eager를 true 했다면 categories 배열도 항상 같이 JOIN해서 가져온다. eager 옵션을 주는 것은 CPU에 부하를 주므로 되도록이면 사용하는 것을 지양해야 한다.  나중에 categories를 가져오고 싶다면 `const questions = await questionRepository.find({ relations:['categories'] })`를 통해서 원하는 함수에서만 사용하여 categories를 가져올 수 있다. 



## Lazy

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToMany} from "typeorm";
import {Question} from "./Question";

@Entity()
export class Category {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @ManyToMany(type => Question, question => question.categories)
    questions: Promise<Question[]>;
}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable} from "typeorm";
import {Category} from "./Category";

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    text: string;

    @ManyToMany(type => Category, category => category.questions)
    @JoinTable()
    categories: Promise<Category[]>;
}
```

Eager가 더 많은 데이터를 가져오려고 한다면 lazy는 relation entity를 나중에 가져올 수 있도록 한다. lazy option을 주지는 않고 `Promise<>`를 사용한다.



```typescript
const questionRepository = connection.getRepository(Question);

const questions = await questionRepository.find();
const question = questions[0];

const categoreis = await question.categories

console.log(categories)
```

lazy 옵션을 주게 되면 JOIN을 사용하지 않기에 N+1 번 쿼리를 날린다. 이것을 lazy loading이라고 한다.

* 장점
  * 초기 로딩 시간을 줄일 수 있다.
  * 자원 소비를 줄일 수 있다.
  * 사용하지 않는 데이터를 결과 객체에 포함시키지 않기때문에 cpu 타임을 절약할 수 있다.
* 단점
  * 그 결과 DB로 더 많은 쿼리를 날리게 된다.
  * 뿐만 아니라 원치 않는 순간에 성능에 영향을 줄 수도 있다.

구체적인 사용 사례로는 sns에서 **댓글 더보기** 버튼을 누르는 경우, eager loading을 사용하는 경우 **댓글 더보기**를 누르지 않았는데도 이미 댓글을 조회해버리기 때문에 성능상 이슈가 생길 수 있다. 이 때는 **댓글 더보기**를 클릭했을 때 댓글 목록을 호출하도록 하는 lazy loading을 사용할 수 있다.



<span style="font-size:2rem;font-style:italic;">**lazy loading은 아직 실험 단계라서 사용하는 것을 권장하지는 않는다!!!!**</span>