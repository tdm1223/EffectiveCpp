# 항목32. public 상속 모형은 반드시 "is-a(...는 ...의 일종이다)"를 따르도록 만들자
## 기본 클래스와 파생 클래스
- **파생 클래스 타입**으로 만들어진 모든 객체는 **기본 클래스 타입**의 객체이지만 **반대는 되지 않는다**.
    - 기본 클래스는 파생 클래스보다 더 **일반적인 개념**을 나타낸다.
    - 파생 클래스는 기본 클래스보다 더 **특수한 개념**을 나타낸다.

```cpp
void eat(const Person& p); // 먹는것은 누구든 함
void study(const Sttudent& s); // 공부는 학생만함

Person p; // p는 Person의 일종
Student s; // s는 Student의 일종

eat(p); // 문제 없음
eat(s); // 문제 없음 s는 Student이고 Student는 Person의 일종

study(s); // 문제 없음
study(p); // 에러. p는 Student가 아니다.
```
- `public` 상속에서만 가능하다.
- [private 상속](/Chapter6/Item39.md)은 의미 자체가 완전히 다르다.
- `public` 상속은 기본 클래스 객체가 가진 **모든 것들**이 파생 클래스 객체에도 그대로 적용된다고 단정한다.

## 클래스 사이에 맺을 수 있는 관계
- is-a 관계
- [has-a 관계](/Chapter6/Item38.md)
- [is-implemented-in-terms-of 관계](/Chapter6/Item39.md)