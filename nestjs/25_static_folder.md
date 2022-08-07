# Static Folder 설정

## 패키지

### 설치

```shell
$ yarn add pug
```

## 디렉토리 구조

<img width="289" alt="image" src="https://user-images.githubusercontent.com/33750210/179892991-e7d3d4e7-1185-4239-be2e-8bc0833f676d.png">

## bootstrap.ts

```typescript
// src/bootstrap.ts

import { join } from 'path/posix';
import { Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { NestFactory } from '@nestjs/core';
import { NestExpressApplication } from '@nestjs/platform-express';

import { AppModule } from './app.module';

export async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  const logger = new Logger('bootstrap');
  const config = app.get(ConfigService);

  const PORT = config.get<string>('app.port');

  // 밑에 3가지 코드를 추가해야 static 파일들을 사용할 수 있다.
  app.useStaticAssets(join(__dirname, '..', 'src', 'static', 'public'));
  app.setBaseViewsDir(join(__dirname, '..', 'src', 'static', 'views'));
  app.setViewEngine('pug');

  await app.listen(PORT, () => {
    if (process.send) {
      process.send('ready');
    }
    logger.log(`✅ Server is listening on http://localhost:${PORT}`);
  });
}

```

  JwtModule은 여태까지 만든 모듈들과 다르다. NestJS는 DynamicModule이라는 것을 제공해주는데, 이것으로 customizing module을 만들 수 있다.

* 참조 : https://docs.nestjs.com/techniques/mvc#model-view-controller
* `  app.useStaticAssets(join(__dirname, '..', 'src', 'static', 'public'));` :  assets 폴더를 사용할 수 있게 한다.
* `app.setBaseViewsDir(join(__dirname, '..', 'src', 'static', 'views'));` :  nestJS가 접근할 views의 경로를 지정한다. 부트스트랩 이후 해당 경로는 절대경로가 되어서 랜더링 경로를 설정할 때 생략해도 된다.
* `app.setViewEngine('pug');` : 사용하고 싶은 랜더링 엔진을 지정한다.

## nest-cli.json

```json
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "tsConfigPath": "tsconfig.json",
    "assets": [
      {
        "include": "views/**/*.pug",
        "outDir": "./dist/src",
        "watchAssets": true
      },
      {
        "include": "public/styles/**/*.css",
        "outDir": "./dist/src",
        "watchAssets": true
      },
      {
        "include": "public/js/**/*.js",
        "outDir": "./dist/src",
        "watchAssets": true
      }
    ]
  }
}
```

`assets` 프로퍼티에 include할 assets을 추가해야 한다.

