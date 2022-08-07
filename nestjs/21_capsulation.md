# Nest Capsulation

## Bad Capsulation

<img src="https://user-images.githubusercontent.com/33750210/147198576-f31ebb32-3bbd-4a9c-88c7-45f5d2edf2ab.png" alt="image" style="zoom:50%;" />

어떤 모듈(AppModule)에서 다른 모듈의 서비스(CatsService)를 사용하고 싶다면 provider에 추가해줘야 한다. 하지만 이 패턴은 좋지않다. 모듈들이 많아지면 provider에서 사용할 수 있는 것들이 계속 추가가 되고 관리가 어려워진다. 

그리고 모듈들이 분리가 되어 있는데, 독립성 원칙을 위반하고 바로 provider에 주입하면 **단일 책임** 원칙이 어긋난다. provider에는 AppModule에서 만든 AppService 같은 객체만 주입해야 하고 CatsService는 CatsModule에서 작성된 것이기 때문에 해당 모듈 자체를 주입하는 것이 좋다.

## Good Capsulation

<img src="https://user-images.githubusercontent.com/33750210/147198461-d04d3625-a626-4015-978f-b4efa415c89d.png" alt="image" style="zoom:50%;" />

**CatsModule**에서 exports에 CatsService를 추가해줘서 은닉화를 해제해 CatsService를 public하게 바꾼다.

<img src="https://user-images.githubusercontent.com/33750210/147198750-f3a7afcd-7450-47c2-9510-2b84eb3ee95f.png" alt="image" style="zoom:50%;" />

**AppModule**에서 **CatsModule**을 주입한다.

## 결론

providers에는 자체적으로 만든 provider를 추가하고 다른 모듈에서 만든 provider는 exports에 추가해서 public으로 만들어주고 다른 모듈에서 imports에 원하는 공급자를 포함한 모듈을 추가한다.





























