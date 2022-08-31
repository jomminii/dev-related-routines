
### 정적 팩토리 메서드
[정적 팩토리 메서드(Static Factory Method)는 왜 사용할까? - Tecoble](https://tecoble.techcourse.co.kr/post/2020-05-26-static-factory-method/)
- 정적 팩토리 메서드란 객체 생성의 역할을 하는 클래스 메서드
- 생성자와의 차이
  - 이름을 가질 수 있다
  - 호출할 때마다 새로운 객체를 생성할 필요가 없다
  - 하위 자료형 객체를 반환할 수 있다
  - 객체 생성을 캡슐화 할 수 있다
    
  - 정적 팩토리 메서드 네이밍 컨벤션
    - from : 하나의 매개 변수를 받아서 객체를 생성
    - of : 여러개의 매개 변수를 받아서 객체를 생성
    - getInstance | instance : 인스턴스를 생성. 이전에 반환했던 것과 같을 수 있음.
    - newInstance | create : 새로운 인스턴스를 생성
    - get[OtherType] : 다른 타입의 인스턴스를 생성. 이전에 반환했던 것과 같을 수 있음.
    - new[OtherType] : 다른 타입의 새로운 인스턴스를 생성.