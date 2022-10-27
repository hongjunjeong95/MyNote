# Jest - Mock Stub

**Ref**

* [Mock Functions](https://jestjs.io/docs/mock-functions)

## Good Exmpale

### product_client.js

```js
class ProductClient {
  fetchItems() {
    return fetch("http://example.com/login/id+password").then((response) =>
      response.json()
    );
  }
}

module.exports = ProductClient;
```

### product_service.js

```javascript
class ProductService {
  constructor(productClient) {
    this.productClient = productClient;
  }

  fetchAvailableItems() {
    return this.productClient
      .fetchItems()
      .then((items) => items.filter((item) => item.available));
  }
}

module.exports = ProductService;

```

* `product_service_no_di.js`와는 다르게 외부에서 Dependency를 주입 받았다.

### stub_product_client.js

```js
class StubProductClient {
  async fetchItems() {
    return [
      { item: "Milk", available: true },
      { item: "Banana", available: false },
    ];
  }
}

module.exports = StubProductClient;
```

`product_service.test.js`가 mock을 생성하지 않도록 ProductClient 클래스의 역할을 수행하는 인터페이스용 클래스 `StubProductClient`를 생성했다. 이는 mock이 아니라 진짜 클래스다.

### product_service.test.js

```js
// network state에 의존하는 코드는 좋지 않다.
// 테스트 코드는 항상 독립적이어야 한다.

const ProductService = require("../product_service");
const StubProductClient = require("./stub_product_client");

describe("ProductService - Stub", () => {
  let productService;

  beforeEach(() => {
    productService = new ProductService(new StubProductClient());
  });

  it("should filter out only available items", async () => {
    const items = await productService.fetchAvailableItems();
    expect(items.length).toBe(1);
    expect(items).toEqual([{ item: "Milk", available: true }]);
  });
});
```

* mock을 생성하지 않고 Stub 인터페이스를 생성해서 ProductService에 주입하는 방법으로 테스팅을 했다. 코드가 훨씬 더 간결해졌다.



## Stub이 아니라 mock을 사용해서 테스팅을 해야만 할 때

### user_client.js

```js
class UserClient {
  login(id, password) {
    return fetch("http://example.com").then((response) => response.json());
  }
}

module.exports = UserClient;
```

### user_service.js

```javascript
class UserService {
  constructor(userClient) {
    this.userClient = userClient;
    this.isLogedIn = false;
  }

  login(id, password) {
    if (!this.isLogedIn) {
      // 서비스 내부에서 fetch를 하게 되면 unit test가 어렵다.
      // 왜냐하면 UserService라는 단위는 network에 항상 의존하기 때문에
      // 그래서 network 통신을 할 때는 별도의 클래스로 분리해야 한다.
      //   return fetch("http://example.com") //
      //     .then((response) => response.json());
      return this.userClient
        .login(id, password) //
        .then((data) => (this.isLogedIn = true));
    }
  }
}
```

*  서비스 내부에서 fetch를 하게 되면 unit test가 어렵다. 왜냐하면 UserService라는 단위는 network에 항상 의존하기 때문이다. 그래서 network 통신을 할 때는 별도의 클래스로 분리해야 한다.

### user_service.test.js

```js
const UserService = require("../user_service");
const UserClient = require("../user_client");

// 호출을 했는지 안 했는지 행동에 대한 테스트를 할 때는 stub만 가지고 테스트 하기에는 무리가 있다.

describe("UserService", () => {
  const login = jest.fn(async () => "success");
  UserClient.mockImplementation(() => {
    return {
      login,
    };
  });
  let userService;

  beforeEach(() => {
    userService = new UserService(new UserClient());
    // login.mockClear();
    // UserClient.mockClear();
  });

  it("calls login() on UserClient when tries to login", async () => {
    await userService.login("abc", "abc");
    expect(login.mock.calls.length).toBe(1);
  });

  it("should not call login() on UserClient again if already logged in", async () => {
    await userService.login("abc", "abc");
    await userService.login("abc", "abc");

    expect(login.mock.calls.length).toBe(1);
  });
});
```

* `jest.config.js`에서 `clearMocks : true`를 해줬기 때문에 `*,mockClear()`는 쓸 필요가 없다.
* `user_service.test.js`에서는 stub을 이용해서 테스트가 불가능하기 때문에 mock을 이용해서 테스트 코드를 짰다.
  *  그 이유는 호출을 했는지 안 했는지 행동에 대한 테스트를 할 때는 stub만 가지고 테스트 하기에는 무리가 있기 때문이다.
