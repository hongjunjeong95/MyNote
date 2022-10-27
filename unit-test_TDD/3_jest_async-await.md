# Jest - Async & Await

**Ref**

* [Testing Asynchronous Code](https://jestjs.io/docs/asynchronous)

## async.js

```js
function fetchProduct(error) {
  if (error == "error") {
    return Promise.reject("network error");
  }
  return Promise.resolve({ item: "Milk", price: 200 });
}

module.exports = fetchProduct;
```

## async.test.js

```javascript
const fetchProduct = require("../async");

describe("Async", () => {
  it("async-done", (done) => {
    fetchProduct().then((item) => {
      expect(item).toEqual({ item: "Milk", price: 200 });
      done();
    });
  });

  it("async-return", () => {
    return fetchProduct().then((item) => {
      expect(item).toEqual({ item: "Milk", price: 200 });
    });
  });

  it("async-await", async () => {
    const product = await fetchProduct();

    expect(product).toEqual({ item: "Milk", price: 200 });
  });

  it("async-resolves", async () => {
    return expect(fetchProduct()).resolves.toEqual({
      item: "Milk",
      price: 200,
    });
  });

  it("async-reject", async () => {
    return expect(fetchProduct("error")).rejects.toBe("network error");
  });
});
```

* 비동기 함수는 done() callback 함수를 이용해서 비동기 요청을 기다린다.
* done 대신 return을 사용해도 된다.
* async & await을 사용해도 된다.
* resolves나 rejects를 이용해서 비동기 테스팅 가능
