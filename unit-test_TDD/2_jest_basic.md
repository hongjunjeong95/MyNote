# Jest - Basic

**Ref**

* [Using Matchers](https://jestjs.io/docs/using-matchers)

## calculator.js

```js
class Calculator {
  constructor() {
    this.value = 0;
  }

  set(num) {
    this.value = num;
  }

  clear() {
    this.value = 0;
  }

  add(num) {
    const sum = this.value + num;
    if (sum > 100) {
      throw new Error('Value can not be greater than 100');
    }
    this.value = sum;
  }

  subtract(num) {
    this.value = this.value - num;
  }

  multiply(num) {
    this.value = this.value * num;
  }

  divide(num) {
    this.value = this.value / num;
  }
}

module.exports = Calculator;
```

## calculator.test.js

```javascript
const Calculator = require("../calculator.js");

describe("Calculator", () => {
  let cal;
  beforeEach(() => {
    cal = new Calculator();
  });

  it("inits with 0", () => {
    expect(cal.value).toBe(0);
  });

  it("sets", () => {
    cal.set(9);

    expect(cal.value).toBe(9);
  });

  it("clear", () => {
    cal.set(9);
    cal.clear();
    expect(cal.value).toBe(0);
  });

  it("adds", () => {
    cal.add(1);
    cal.add(2);

    expect(cal.value).toBe(3);
  });

  it("add should throw an error if value is greater than 100", () => {
    expect(() => {
      cal.add(101);
    }).toThrow("Value can not be greater than 100");
  });

  it("subtracts", () => {
    cal.subtract(1);

    expect(cal.value).toBe(-1);
  });

  it("multiplies", () => {
    cal.set(5);
    cal.multiply(4);

    expect(cal.value).toBe(20);
  });

  describe("divides", () => {
    if (
      ("0 / 0 === NaN",
      () => {
        cal.divide(0);
        expect(cal.value).toBe(NaN);
      })
    );
    if (
      ("1 / 0 === Infinity",
      () => {
        cal.set(1);
        cal.divide(0);
        expect(cal.value).toBe(Infinity);
      })
    );

    if (
      ("4 / 4 === 1",
      () => {
        cal.set(4);
        cal.divide(4);

        expect(cal.value).toBe(1);
      })
    );
  });
});
```

* test : 한 개의 테스트 코드 실행
* describe : 여러 개의 테스트 로직을 묶음
  * it : describe 안에서 한 개의 테스트 코드 실행
  * beforeEach : 테스트 코드가 실행되기 전에 실행될 함수
* expect : 인자로 함수를 받으며 `toBe` 등으로 체이닝을 하여 결과 값을 기대
  * toBe : 예상되는 함수의 결과 값을 인자로 받음
    * NaN, Infinity, number
  * toThrow : 예상되는 함수의 에러 메시지
