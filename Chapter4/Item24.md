# 항목 24. 타입 변환이 모든 매개변수에 대해 적용되어야 한다면 비멤버 함수를 선언하자
- 클래스에서 암시적 타입 변환을 지원하는 것은 좋지않다.
  - 예외로는 숫자 타입을 만들때가 있다.
## 유리수를 나타내는 클래스
```cpp
class Rational{
public:
    Rational(int numerator = 0, int denominator = 1);
    int numerator() const;
    int denominator() const;
}
```
- 생성자에 일부러 `explicit`를 붙이지 않았다.
- `int`에서 `Rational`로의 **암시적 변환**을 허용한다.

## 곱셈 지원
```cpp
class Rational{
public:
    Rational(int numerator = 0, int denominator = 1);
    int numerator() const;
    int denominator() const;
    const Rational operator* (const Rational& rhs) const; // 멤버함수로 추가!
};

Rational oneHalf(1, 2);
Rational oneEight(1, 8);
Rational result = oneHalf * oneEight; // 성공
```
- 멤버 함수로 추가하여 유리수 곱셈을 간단하게 수행할 수 있다.

### 혼합형 수치연산
```cpp
result = oneHalf * 2; // 성공 result = oneHalf.operator*(2)
result = 2 * oneHalf; // 에러 result = 2.operator*(onehalf)
```
- 성공하는 연산은 2라는 값을 암시적 변환을 통해 `Rational` 임시 객체를 생성한다.
    - `Rational` 클래스에 `explicit`을 붙였다면 두 연산 모두 실패하게 된다.
- **혼합형 수치연산**을 지원하지 못하는 문제가 생기게 된다.
- 곱셈은 교환법칙이 성립해야 하나 성립하지 않고 있다.
- **암시적 타입 변환**에 대해 매개변수가 먹혀들려면 **매개변수 리스트**에 들어 있어야만 한다.
    - 호출되는 멤버 함수를 갖고 있는 객체에 해당하는 암시적 매개변수에는 암시적 변환이 먹히지 않는다.

## 혼합형 수치연산 지원하기
- `operator*`를 **비멤버 함수**로 만들어 컴파일러 쪽에서 **모든 인자**에 대해 **암시적 타입 변환**을 수행하도록 한다.
```cpp
class Rational{};

const Rational operator*(const Rational* lhs, const Rational* rhs)
{
    return Rational (lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}
```
- 멤버 함수의 반대는 프렌드 함수가 아니라 비멤버 함수이다.