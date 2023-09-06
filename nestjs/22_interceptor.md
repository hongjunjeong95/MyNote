# Interceptor

Middleware와 유사해보이지만 interceptor는 req와 res에 헤더나 데이터를 추가해줄 수 있다.

<img src="https://user-images.githubusercontent.com/33750210/147202578-f0392e3e-75c0-448a-a5c5-e1d818e38814.png" alt="image" style="zoom:50%;" />

`/cats`로 get 요청을 했을 때 인터셉터가 res를 client에게 보낼 때

<img src="https://user-images.githubusercontent.com/33750210/147202664-3a1b3383-55a6-47c5-89e2-13067a4dba54.png" alt="image" style="zoom:40%;" />

```json
{
  "success": true,
  "data":{
    "cats":"get all cat api"
  }
}
```

로 반환한다.