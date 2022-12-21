# Jest - Command

**Ref**

* [Jest Installation](https://jestjs.io/docs/getting-started)

## Commands

* `yarn run test` : 테스트 실행
* `jest --coverage` : `jest.config.js`에서 `collectCoverage : false`면 `jest --coverage`를 통해서 coverage를 확인할 수 있음

## Watch all test

```json
"scripts":{
  "test": "jest --watchAll"
}
```

<img src="https://user-images.githubusercontent.com/92770273/146470527-13b59f8c-dc63-4e13-acc6-2f9b43e463bb.png" alt="image" style="zoom:50%;" />

## Watch test

```json
"scripts":{
  "test": "jest --watch"
}
```

내가 커밋 하려는 코드만 테스트하고 싶을 때는 `jest --watch`와 git 파일을 생성하면 된다.

## Watch test --verbose

```json
"scripts":{
  "test": "jest --watch --verbose"
}
```

<img src="https://user-images.githubusercontent.com/92770273/146473035-1659f875-9504-40f1-b318-075b0d502315.png" alt="image" style="zoom:50%;" />

`--verbose`를 추가하면 describe와 it에 대한 로그도 출력
