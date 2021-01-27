# 항목 21. 함수에서 객체를 반환해야 할 경우에 참조자를 반환하려고 들지 말자
- [값에 의한 전달의 효율](/Chapter4/Item20.md)을 알았다고 모두 참조에 의한 전달로 바꾸려는것은 좋지않다.
- 참조자는 **존재하는 객체**에 붙는 **다른 이름**일 뿐이다.

```cpp
class Rational{
public:
    Rational(int numerator = 0, int denominator = 1);
private:
    int n,d;
friend const Rational operator*(const Rational& lhs, const Rational& rhs);
};

Rational a(1, 2);
Rational b(3, 5);
Rational c = a * b;
```
- 위 클래스에서 `operator*`가 참조자를 반환하도록 만들어졌다면?
    - 함수가 반환하는 참조자는 반드시 이미 **존재하는** `Rational` 객체의 참조자여야 한다.
- 반환될 객체는 어디 있는가? 
    - 참조자를 반환하려면 유리수 객체를 **직접 생성**해야만 한다.

### 함수 수준에서 새로운 객체 만드는 방법
- **함수 수준**에서 **새로운 객체**를 만드는 방법은 스택에 만드는 것, 힙에 만드는 것 두가지 뿐이다.
1. 스택
```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
    return result;
}
```
- `result`는 **지역 객체**이다. (함수가 끝날때 **소멸**된다)
- 지역 객체를 반환한다면?
    - 반환되는것은 `Rational` 객체였던 소멸자가 호출된 **바이트 덩어리**가 반환될 것이다.
    
2. 힙
```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational *result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    return *result;
}
```
- `new`로 생성한 객체는 어디서 `delete`될것인가?
    - 누가 `delete`로 삭제해줄 것인지 알 수 없다.

### Rational 객체를 정적 객체로 함수 안에 정의해 놓고 이것의 참조자를 반환한다면?
- **스레드 안정성**의 문제
- `operator==`연산자 오버로딩시 항상 같은 값을 반환할 수 있다. (정적 객체)

### 정적 데이터를 배열로 쓴다면?
- 배열의 크기를 먼저 정해야 한다.
    - 작으면 반환값을 저장할 공간이 부족하고 크다면 수행 성능이 떨어진다.
    - 적당한 크기를 잡아야 한다. (불가능)

### BEST 코드
```cpp
inline const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```
- **새로운 객체**를 반환하게 만든다.
- 객체 생성, 소멸의 비용이 들어가지만 올바른 동작에 지불되는 작은 비용일 뿐이다.