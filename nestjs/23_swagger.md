# Swagger

**Ref**

* https://docs.nestjs.com/openapi/introduction

## Install

```shell
yarn add @nestjs/swagger swagger-ui-express
```

## Bootstrap

```typescript
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('Cats example')
    .setDescription('The cats API description')
    .setVersion('1.0')
    .addTag('cats')
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}
bootstrap();
```

## users.controller.ts

```typescript
@ApiTags('users')
@Controller('')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @ApiOperation({ summary: 'Sign up' })
  @ApiResponse({
    status: 200,
    description: 'success',
    type: CreateAccountOutput,
  })
  @ApiResponse({
    status: 500,
    description: 'Server Error...',
  })
  @Post('/signup')
  async createAccount(
    @Body() body: CreateAccountInput,
  ): Promise<CreateAccountOutput> {
    const createAccountInput: CreateAccountInput = {
      ...body,
    };
    return this.usersService.createAccount(createAccountInput);
  }
}
```

* @ApiTags : controller 별로 구획을 나눈다.
* @ApiOperation으로 해당 api에 대한 description을 준다.
* @ApiResponse
  * Status에 따라 @Api
* parameter 설명은 @Body, @Param, @Query로 나눈다.
  * @ApiBody(), @ApiParam(), @ApiQuery()로 설명
* 기타 : @ApiHeader(), @ApiHeaders(), @ApiCookieAuth(), @ApiBearerAuth(), @ApiOkResponse() 등등이 있음

## user.entity.ts

```typescript
@Entity()
export class User extends CommonEntity {
  @ApiProperty({
    example: 'lewis@gmail.com',
    description: 'email',
    required: true,
    type: String,
  })
  @Column({ unique: true, type: 'text' })
  @IsEmail()
  email: string;
  
  @ApiProperty({
    example: UserRole.Consumer,
    description: 'User Role',
    required: true,
    enum: UserRole,
  })
  @Column({ type: 'enum', enum: UserRole, default: UserRole.Consumer })
  @IsEnum(UserRole)
  role: UserRole;
  
  @ApiProperty({
    example: 'products',
    description: 'products',
    required: true,
    type: [Product],
  })
  @OneToMany(() => Product, (product) => product.provider)
  products: Product[];
}
```

Entity에서 @ApiProperty로 테이블의 속성을 정의

## Swagger Security

### Install

```shell
yarn add express-basic-auth
```

### bootstrap

```typescript
import * as expressBasicAuth from 'express-basic-auth';

app.use(
    ['/docs', '/docs-json'],
    expressBasicAuth({
      challenge: true,
      users: { [process.env.SWAGGER_USER]: process.env.SWAGGER_PASSWORD },
    }),
  );
```