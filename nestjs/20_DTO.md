# Nest DTO

## **Interface VS Class For DTO**

DTO 는 Interface나 Class를 사용해서 만들면 됩니다. 하지만 Class가 더 선호됩니다. 선호되는 이유는 ?

TypeScript 인터페이스를 사용하거나 간단한 클래스를 사용하여 DTO 스키마를 결정할 수 있다. 흥미롭게도 여기에서 Class를 사용하는 것이 좋다. 왜? 클래스는 JavaScript ES6 표준의 일부이므로 컴파일 된 JavaScript에서 실제 엔티티로 유지된다. 반면에 TypeScript 인터페이스는 트랜스 파일 중에 제거되므로 Nest는 런타임에서 참조 할 수 없다.

이것은 파이프와 같은 기능을 런타임에서 사용할 수 있기 때문에 런타임에서 사용 될 수 있는게 중요하다.