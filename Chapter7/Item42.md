# 항목 42. typename의 두 가지 의미를 제대로 파악하자
### class와 typename의 차이
```cpp
template<class T> class Base;
template<typename T> class Base;
```
- 아무런 차이가 없다.
- **템플릿 매개변수를 선언**하는 경우에 둘은 완전히 같은 의미를 가진다.

### 의존 이름과 비의존 이름
1. 의존 이름(dependent name)
    - 템플릿 매개변수에 따라 달라지는 타입
    - 의존 이름이 어떤 클래스 안에 중첩되어 있는 경우 **중첩 의존 타입 이름**(nested dependent typename)이라고 한다.
2. 비의존 이름(non-dependent name)
    - 템플릿 매개변수가 어떻든 상관없는 타입

### 코드 안에 중첩 의존 이름이 존재할때
```cpp
template<typename C>
void print(const C& container)
{
    C::const_iterator * x;
}
```
- `c::const_iterator`에 대한 포인터 지역 변수 x를 선언하는것 같다.
  - `C::const_iterator`가 **타입**이라는 사실을 알고 있을때만
1. `C::const_iterator`가 타입이 아니라면?
2. `const_iterator`라는 이름을 가진 정적 데이터 멤버가 C에 있다면?
3. `x`가 다른 전역 변수의 이름이라면?
    - 그냥 곱셈이 된다!

### typename을 사용해야 할 때
- `C`의 정체가 무엇인지 다른 곳에서 알려주지 않으면 `C::const_iterator`가 진짜 타입인지 아닌지를 알아낼 방법은 없다.
- `C++` 컴파일러에게 `C::const_iterator`가 타입이라고 알려 줄 때 앞에다가 `typename`이라는 키워드를 붙여 놓는다.

```cpp
template<typename C>
void print(const C& container)
{
    typename C::const_iterator * x;
}
```
- 템플릿 안에서 **중첩 의존 이름**을 참조할 경우에는 이름 앞에 `typename` 키워드를 붙여준다. (예외도 존재한다)

### 중첩 의존 이름 앞 typename 키워드 예외
- 중첩 의존 타입 이름이 **기본 클래스의 리스트**에 있거나 **멤버 초기화 리스트 내의 기본 클래스 식별자**로서 있을 경우에는 `typename`을 붙여 주면 안 된다.
```cpp
template<typename T>
class Derived: public Base<T>::Nested{ // 상속되는 기본 클래스 리스트 : typename 쓰면 안됨
public:
    explicit Derived(int x) 
     : Base<T>::Nested(x) // 멤버 초기화 리스트에 있는 기본 클래스 식별자 : typename 쓰면 안됨
    {
        typename Base<T>::Nested temp;
    }
};
```

### typedef typename의 활용
```cpp
template<typename IterT>
void Test(IterT iter)
{
    typedef typename std::iterator_traits<IterT>::value_type value_type;
    value_type temp(*iter);
}
```
- `typename`을 붙여주는 변수에 대해 변수명이 정말 복잡하거나 긴 상황이 발생할 수 있다.
- `typedef`이름을 만들때 `typedef typename`이 연속적으로 나오나 전혀 이상한 표현이 아니다.
- [C++ 표준의 특성정보 클래스](/Chapter7/Item47.md)를 사용한 것 뿐이다.
- 중첩 의존 타입 이름을 참조하는 데 지켜야 할 규칙 때문에 생긴 부산물이고 논리적으로 문제가 없다.
