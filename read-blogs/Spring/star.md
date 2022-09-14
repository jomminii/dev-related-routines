
### [SOLID 원칙 - 기계인간 John Grib](https://johngrib.github.io/wiki/jargon/solid/)

예시가 많이 없어서 그런지 뭔가 더 헷갈려진 느낌... 예시 많은데 찾아보고 다시 한 번 봐야겠다.

### [Spring Framework에서 생성자 주입을 권장하는 이유! - 코딩하는 흑구](https://sas-study.tistory.com/456)
- new 로 인스턴스화한다고 의존주입이라고 표현하진 않음. 프레임워크의 동작 내에서 이루어져야함.
- 필드 주입이나 수정자 주입은 순환참조 오류 발생할 수 있음
- 생성자 주입을 써야함