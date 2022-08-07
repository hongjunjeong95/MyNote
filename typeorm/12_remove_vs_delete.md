# remove vs delete

**Ref**

* https://github.com/typeorm/typeorm/blob/master/docs/repository-api.md

- remove : 무조건 존재하는 아이템을 remove 메소드를 이용해서 지워야한다 그러지 않으면 에러가 발생한다.(404 Error)
- delete: 만약 아이템이 존재하면 지우고 존재하지 않으면 아무런 영향이 없다. 이러한 차이 때문에 remove를 이용하면 하나의 아이템을 지울 때 두번 데이터베이스를 이용해야 하기 때문에 (아이템 유무 + 지우기) 데이터베이스에 한번만 접근해도 되는 delete 메소드를 사용하자.

## 
