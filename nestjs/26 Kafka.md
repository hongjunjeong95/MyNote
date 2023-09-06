# Kafka

## Message Pattern

### API Server

#### subtasks.module.ts

```js
// subtasks.module.ts

import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

export { SubtasksService } from './subtasks.service';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'SUBTASKS_SERVICE',
        transport: Transport.KAFKA,
        options: {
          client: {
            clientId: 'subtask',
            brokers: ['localhost:9092'],
          },
          consumer: {
            groupId: 'subtask-consumer',
          },
        },
      },
    ]),
  ],
  providers: [
    SubtasksService,
  ],
  exports: [SubtasksService],
})
export class SubtasksModule {}
```

1. 내가 쓰고 싶은 서비스 서버의 **name**을 지정하고 해당 서비스 서버에서 설정된 **consumer groupId**를 맞춰준다. 

#### subtasks.service.ts

```js
// subtasks.service.ts

import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Injectable()
export class SubtasksService {
  constructor(
    // repositories
    @Inject(SubtasksRepositoryInterfaceToken)
    private subtasksRepository: SubtaskRepositoryInterface<
      Subtask,
      SubtaskCreateInput
    >,

    // Kafka
    @Inject('SUBTASKS_SERVICE') private readonly subtasksClient: ClientKafka,
  ) {}

  async queueUploadResult(params: SubtaskIdDto & PutSubtaskResultDto) {
    const { subtaskId, isWorkerComplete } = params;
    await this.findByIdOrThrow(subtaskId);
    await this.uploadResultByKafka(subtaskId, isWorkerComplete);

  
  }

  // subtaskClient이니깐 api 서버에서 subtask.service에 있어야 하지 않나요?
  private async uploadResultByKafka(
    subtaskId: number,
    isWorkerComplete?: boolean,
  ) {
    await this.subtasksRepository.updateMetadata(subtaskId, {
      uploading: true,
    });

    // message 방식에서는 send를 사용한다.
    this.subtasksClient
      .send(
        QueueTopics.UPLOAD_SUBTASK_RESULT_TOPIC,
        new UploadSubtaskResultEvent(subtaskId, isWorkerComplete),
      )
      .subscribe(async (res) => {
        console.log(res);
        await this.subtasksRepository.updateMetadata(subtaskId, {
          uploading: false,
        });
      });
  }

  async init() {
    [QueueTopics.UPLOAD_SUBTASK_RESULT_TOPIC].forEach((key) => {
      /*
      	service server가 보내는 반환 값을 해당 토픽으로부터 subscribe할 수 있게 해준다.
      */
      this.subtasksClient.subscribeToResponseOf(key); 
    });
    await this.subtasksClient.connect();
  }

  async destroy() {
    await this.subtasksClient.close();
  }
};
```

1. `@Inject('SUBTASKS_SERVICE') private readonly subtasksClient: ClientKafka,` : 서비스를 호출하는 모듈에서 **ClientsModule**을 호출하였다면 name으로 지정한 서비스를 사용할 수 있다. 타입은 **ClientKafka**다.

2. ```js
   this.subtasksClient
     .send(
       QueueTopics.UPLOAD_SUBTASK_RESULT_TOPIC,
       new UploadSubtaskResultEvent(subtaskId, isWorkerComplete),
     )
     .subscribe(async (res) => {
       console.log(res);
       await this.subtasksRepository.updateMetadata(subtaskId, {
         uploading: false,
       });
     });
   ```

   1. `send` 메소드를 사용하면 메시지 방식으로 publish 할 수 있다. 메시지 방식은 이벤트 방식과 다르게 response를 기다린다. **SUBTASK_SERVICE**의 컨슈밍 로직이 끝나고 **return** 값을 주면 `subscribe` 메소드가 실행되고 해당 반환 값이 res parameter에 담기고 **subscribe** 콜백 함수가 실행된다.



### Subtask Service Server

#### main.ts

```js
// main.ts

import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';

import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.KAFKA,
    options: {
      client: {
        brokers: ['localhost:9092'],
      },
      consumer: {
        groupId: 'subtask-consumer',
      },
    },
  });
  app.listen();
}
bootstrap();
```

1. `await NestFactory.createMicroservice(AppModule, {` : `createMicroservice` 메소드를 사용한다.
2. 두 번째 파라미터에 옵션 값을 설정해주는데, 이 서버의 카프카 옵션 값을 설정해준다.

#### app.controller.ts

```js
// app.controller.ts

import { Controller } from '@nestjs/common';
import {
  Ctx,
  KafkaContext,
  MessagePattern,
  Payload,
} from '@nestjs/microservices';

import { UploadSubtaskResultEvent } from './events/upload-subtask-result';
import { SubtasksService } from './app.service';

import { QueueTopics } from '@the-datahunt/datahunt-core/queues/topics';

@Controller()
export class SubtasksController {
  constructor(
    //
    private subtasksService: SubtasksService,
  ) {}

  @MessagePattern(QueueTopics.UPLOAD_SUBTASK_RESULT_TOPIC)
  uploadSubtaskResult(
    @Payload() message: UploadSubtaskResultEvent,
    @Ctx() context: KafkaContext,
  ) {
    const originalMessage = message;
    return this.subtasksService.uploadSubtaskResult(originalMessage);
  }
}
```

1. `@MessagePattern(QueueTopics.UPLOAD_SUBTASK_RESULT_TOPIC)` : MessagePattern decorator를 사용하여 API 서버가 보내는 메시지 토픽을 consuming 한다.
2. `return this.subtasksService.uploadSubtaskResult(originalMessage);` : return을 하면 **Subtask Service Server**를 subscribe하고 있는 API 서버가 데이터를 받는다.



## Event Pattern

### API Server

#### subtasks.module.ts

```js
// subtasks.module.ts

import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

export { SubtasksService } from './subtasks.service';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'SUBTASKS_SERVICE',
        transport: Transport.KAFKA,
        options: {
          client: {
            clientId: 'subtask',
            brokers: ['localhost:9092'],
          },
          consumer: {
            groupId: 'subtask-consumer',
          },
        },
      },
    ]),
  ],
  providers: [
    SubtasksService,
  ],
  exports: [SubtasksService],
})
export class SubtasksModule {}
```

1. 내가 쓰고 싶은 서비스 서버의 **name**을 지정하고 해당 서비스 서버에서 설정된 **consumer groupId**를 맞춰준다. 

#### subtasks.controller.ts

```js
// subtasks.controller.ts

import {
  Controller,
  Get,
  OnModuleInit,
  OnModuleDestroy,
} from '@nestjs/common';

import {
  SubtasksService,
} from '@the-datahunt/datahunt-core';
import { Subtask } from '@the-datahunt/datahunt-core/subtasks/subtask.entity';

@Controller()
export class AdminsSubtasksController implements OnModuleInit, OnModuleDestroy {
  @Get()
  async getHello(){
    return this.subtasksService.getHello();
  }
}
```

#### subtasks.service.ts

```js
// subtasks.service.ts

import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Injectable()
export class SubtasksService {
  constructor(
    // repositories
    @Inject(SubtasksRepositoryInterfaceToken)
    private subtasksRepository: SubtaskRepositoryInterface<
      Subtask,
      SubtaskCreateInput
    >,

    // Kafka
    @Inject('SUBTASKS_SERVICE') private readonly subtasksClient: ClientKafka,
  ) {}

  async queueUploadResult(params: SubtaskIdDto & PutSubtaskResultDto) {
    const { subtaskId, isWorkerComplete } = params;
    await this.findByIdOrThrow(subtaskId);
    await this.uploadResultByKafka(subtaskId, isWorkerComplete);

  
  }

  // subtaskClient이니깐 api 서버에서 subtask.service에 있어야 하지 않나요?
  private async uploadResultByKafka(
    subtaskId: number,
    isWorkerComplete?: boolean,
  ) {
    await this.subtasksRepository.updateMetadata(subtaskId, {
      uploading: true,
    });

    // message 방식에서는 send를 사용한다.
    this.subtasksClient.emit(
      QueueTopics.UPLOAD_SUBTASK_RESULT_TOPIC,
      new UploadSubtaskResultEvent(subtaskId, isWorkerComplete),
    );
  }

  async init() {
    await this.subtasksClient.connect();
  }

  async destroy() {
    await this.subtasksClient.close();
  }
};
```

1. `@Inject('SUBTASKS_SERVICE') private readonly subtasksClient: ClientKafka,` : 서비스를 호출하는 모듈에서 **ClientsModule**을 호출하였다면 name으로 지정한 서비스를 사용할 수 있다. 타입은 **ClientKafka**다.

2. ```js
   // message 방식에서는 send를 사용한다.
   this.subtasksClient.emit(
     QueueTopics.UPLOAD_SUBTASK_RESULT_TOPIC,
     new UploadSubtaskResultEvent(subtaskId, isWorkerComplete),
   );
   ```

   1. `emit` 메소드를 사용하면 이벤트 방식으로 publish 할 수 있다. 이벤트 방식은 메시지 방식과 다르게 topic을 보내놓고 신경쓰지 않는다.



### Subtask Service Server

#### main.ts

```js
// main.ts

import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';

import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.KAFKA,
    options: {
      client: {
        brokers: ['localhost:9092'],
      },
      consumer: {
        groupId: 'subtask-consumer',
      },
    },
  });
  app.listen();
}
bootstrap();
```

1. `await NestFactory.createMicroservice(AppModule, {` : `createMicroservice` 메소드를 사용한다.
2. 두 번째 파라미터에 옵션 값을 설정해주는데, 이 서버의 카프카 옵션 값을 설정해준다.

#### app.controller.ts

```js
import { Controller } from '@nestjs/common';
import {
  Ctx,
  EventPattern,
  KafkaContext,
  Payload,
} from '@nestjs/microservices';

import { UploadSubtaskResultEvent } from './events/upload-subtask-result';
import { SubtasksService } from './app.service';

import { QueueTopics } from '@the-datahunt/datahunt-core/queues/topics';

@Controller()
export class SubtasksController {
  constructor(
    //
    private subtasksService: SubtasksService,
  ) {}

  @EventPattern(QueueTopics.UPLOAD_SUBTASK_RESULT_TOPIC)
  uploadSubtaskResult(
    @Payload() message: UploadSubtaskResultEvent,
    @Ctx() context: KafkaContext,
  ) {
    const originalMessage = message;
    return this.subtasksService.uploadSubtaskResult(originalMessage);
  }
}
```

1. `@EventPattern(QueueTopics.UPLOAD_SUBTASK_RESULT_TOPIC)` : EventPattern decorator를 사용하여 API 서버가 보내는 이벤트 토픽을 consuming 한다.
