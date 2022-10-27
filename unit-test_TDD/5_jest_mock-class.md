# Jest - Mock Class

**Ref**

* [Mock Functions](https://jestjs.io/docs/mock-functions)

## Bad Exmpale

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

### product_service_no_di.js

```javascript
const ProductClient = require("./product_client");
class ProductService {
  constructor() {
    this.productClient = new ProductClient();
  }

  fetchAvailableItems() {
    return this.productClient
      .fetchItems()
      .then((items) => items.filter((item) => item.available));
  }
}

module.exports = ProductService;
```

* ProductService가 ProductClient에 의존성을 가지면서 클래스 내부에서 class를 할당받았다. 이것은 DI 규칙에 어긋난다. 의존성은 외부에서 주입받아야 한다.
* 여기서 테스팅 코드를 짤 때 ProductService의 unit test 독립성을 보장하기 위해 ProductClient와 ProductClient가 사용하는 fetchItems()의 network 통신의 영향을 받으면 안된다.

### product_service_no_di.test.js

```js
// network state에 의존하는 코드는 좋지 않다.
// 테스트 코드는 항상 독립적이어야 한다.

const ProductService = require("../product_service_no_di");
const ProductClient = require("../product_client");
jest.mock("../product_client");

describe("ProductService", () => {
  const fetchItems = jest.fn(async () => [
    {
      item: "Milk",
      available: true,
    },
    {
      item: "Banana",
      available: false,
    },
  ]);
  ProductClient.mockImplementation(() => {
    return {
      fetchItems,
    };
  });
  let productService;

  beforeEach(() => {
    productService = new ProductService();
  });

  it("should filter out only available items", async () => {
    const items = await productService.fetchAvailableItems();
    expect(items.length).toBe(1);
    expect(items).toEqual([{ item: "Milk", available: true }]);
  });
});
```

* `jest.mock("../product_client")` : 해당 path에 있는 함수 및 클래스를 mock 한다.
* `const fetchItems = jest.fn( ... )` : ProductClient의 method를 mock 한다.
* `ProductClient.mockImplementation` : 해당 mock에서 구현되서 사용될 함수를 인자로 받는다. `ProductClient`는 클래스이므로 key(func) : value(구현) 을 반환하는 함수를 `mockImpletation`에 정의한다.
