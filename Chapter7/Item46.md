# 항목 46. 타입 변환이 바람직할 경우에는 비멤버 함수를 클래스 템플릿 안에 정의해 두자
- 모든 매개변수에 대해 암시적 타입 변환이 되도록 만들기 위해서는 [비멤버 함수를 사용](/Chapter4/Item24.md)해야 한다.

### Rational 클래스
```cpp
template<typename T>
class Rational {
public:
    Rational(const T& num = 0, const T& deno = 1) : numerator(num), denominator(deno) {}
    const T Numerator() const
    {
        return numerator;
    }
    const T Denominator() const
    {
        return denominator;
    }
private:
    T numerator;
    T denominator;
};

template<typename T>
const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs)
{
    // ...
}

Rational<int> oneHalf(1,2);
Rational<int> result = oneHalf * 2; // 컴파일 에러!
```
- 템플릿의 경우 어떤 함수를 호출하려는지 컴파일러는 알지 못한다.
- 컴파일러가 아는 것은 Rational<T> 타입의 매개변수를 두 개받아들이는 `operator*`라는 이름의 함수를 **인스턴스로 만들어야 하는 것**이다.
    - 컴파일러는 `T`가 무엇인지 알지 못한다.
- 첫 번째 매개변수(oneHalf)는 Rational<int> 타입으로 `T`는 `int`일 수 밖에 없다.
- 두 번째 매개변수(2)는 Rational<T> 타입으로 선언되어 있고 `operator*`에 넘겨진 두 번째 매개변수는 `int` 타입이다.
    - `T`의 타입을 유추하기 쉽지 않다.
- 컴파일러가 생성자를 써서 `2`를 `Rational<int>`로 변환하고 `T`가 `int`라고 유추할 수 있지 않을까?
    - 템플릿 인자 추론 과정에서는 **암시적 타입 변환**은 고려되지 않는다.

### 컴파일을 성공하려면
- 클래스 템플릿 안에 **프렌드 함수**를 넣는다.
    - 함수 템플릿으로서의 성격을 주지 않고 특정한 함수 하나를 나타낼 수 있다.
- 클래스 템플릿은 템플릿 인자 추론 과정에 좌우되지 않으므로 `T`의 정확한 정보는 `Rational<T>` 클래스가 **인스턴스화**될 당시에 알 수 있다.
```cpp
template<typename T>
class Rational{
public:
    friend const Rational operator*(const Rational& lhs, const Rational& rhs); // operator* 함수 선언
};

template<typename T>
const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs) // operator* 함수 정의
{
    // ...
}
```
- 컴파일은 성공하나 링크가 되지 않는다.

### 링크에 성공하려면
- `operator*`는 `Rational` 안에 선언만 되어 있고 정의는 되어 있지 않다.
- `operator*` 함수의 **본문**을 **선언부**와 붙인다.
```cpp
template<typename T>
class Rational{
public:
    friend const Rational operator*(const Rational& lhs, const Rational& rhs)
    {
        return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
    }
};
```
- 컴파일, 링크, 실행 모두 가능하다.