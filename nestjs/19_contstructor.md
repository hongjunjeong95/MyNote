# Nest Constructor

TS에서는 원래 밑에 처럼 constructor 위에 타입이 지정된 변수를 선언하고 생성자 안에 this로 값을 할당해줘야 한다.

<img src="https://user-images.githubusercontent.com/33750210/147050861-70160ab9-4329-4104-a7f8-9723e1708d0e.png" alt="image" style="zoom:50%;" />

하지만 접근 제한자(public, protected, private)을 생성자 파라미터에 선언하면 접근 제한자가 사용된 파라미터는 암묵적으로 클래스 프로퍼티로 선언된다. 여기서 private이 선언되었기 때문에 dishService 프로퍼티는 DishResolver 클래스 내부에서만 사용 가능하다.

<img src="https://user-images.githubusercontent.com/33750210/147051004-33f0496e-98f4-49f0-923b-80dc7f3f36cd.png" alt="image" style="zoom:50%;" />

