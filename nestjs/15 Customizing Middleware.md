# Customizing Middleware (Class)



  Middleware는 라우터가 클라이언트의 요청을 처리하기 전에 먼저 작동하는 함수다. Express의 미들웨어와 똑같다고 보면 된다. nestjs에서는 글로벌하게 또는 method 별로 router별로 미들웨어를 적용할 수 있다. 여기서는 커스터마이징한 미들웨어를 'posts' 라우터에 적용할 것이다.



### logger.middleware.ts

```typescript
// src/middlewares/logger.middleware.ts

import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```

  logger.middleware.ts에서 처럼 미들웨어를 직접 만들 수 있다. `NestMiddleware`를 implements 하면 use 함수를 작성할 수 있는데, 형식은 express와 똑같다.



### app.module.ts

```typescript
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: '/posts', method: RequestMethod.ALL });
  }
}
```

  `LoggerMiddleware`를 사용하는 방법은 `AppModule`이 `NestModule`을 구현하고 conigure method를 작성하면 된다. apply method에서 적용할 미들웨어 클래스를 전달하고 `.forRoutes`에서 해당 미들웨어를 적용할 라우터와 메소드를 추가한다. `{ path: '', method: '' }`  형식 말고도 controller class 자체를 전달할 수 있다.



###### 참조

* [Customizing Middleware (Class)](https://docs.nestjs.com/middleware#middleware)



###### 소스 코드

* https://github.com/insomenia/edu-nestjs-example/commit/b83d4ca103893714946ec23020f49e4be0f5e596