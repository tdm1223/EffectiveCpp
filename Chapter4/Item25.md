# 항목 25. 예외를 던지지 않는 swap에 대한 지원도 생각해 보자
## SWAP 함수
- 초창기 `STL` 부터 포함된 객체의 값을 맞바꾸는 함수
- [예외 안전성 프로그래밍](/Chapter5/Item29.md)에 없어선 안될 감초 역할
- [자기대입 현상](/Chapter2/Item11.md)의 가능성에 대처하기 위한 대표적인 메커니즘
```cpp
namespace std 
{
    template<typename T>
    void swap(T& a, T& b)
    {
        T temp(a);
        a = b;
        b = temp;
    }
}
```
- 표준에서 기본적으로 제공하는 `swap`은 **복사**만 제대로 지원하는 타입이라면 어떤 타입의 객체이든 **맞바꾸기 동작**을 수행해 준다.
- **한 번 호출**에 **세 번 복사**가 일어난다.

## 표준 SWAP의 단점
- 복사하면 손해를 보는 타입들 중 으뜸은 아마도 다른 타입의 실제 데이터를 가리키는 포인터가 주성분인 타입을 것이다.
- 이러한 개념을 설계의 미학으로 끌어올려 사용하는 기법이 [pimpl 관용구](/Chapter5/Item31.md)이다.
```cpp
class WidgetImpl {
private:
    int a, b, c;
    std::vector<double> v;
};

class Widget {
public:
    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs)
    {
        *pImpl = *(rhs.pImpl);
    }
private:
    WidgetImpl* pImpl;
};
```
- `Widget`객체를 직접 맞바꾼다면 `pImpl` 포인터만 바꾸면 되지만 표준 `swap`은 **세 번 복사**를 하게 된다.
  - `Widget` 객체와 `WidgetImpl`객체 세 개를 복사하게 된다.
  - 매우 비효율적이다.
- `Widget`객체를 맞바꿀 때는 `pImpl` 포인터만 맞바꾸라고 `std::swap`에다가 알려준다.
    - `std::swap`을 `Widget`에 대해 특수화 한다.

### 수정
1. `Widget` 안에 `swap`이라는 **멤버 함수**를 선언한다.
2. 함수가 실제 **맞바꾸기를 수행**하도록 만든다.
3. `std::swap`의 특수화 함수에게 그 멤버 함수를 호출하는 일을 맡긴다.

```cpp
class WidgetImpl {
private:
    int a, b, c;
    vector<double> v;
};

class Widget {
public:
    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs)
    {
        *pImpl = *(rhs.pImpl);
    }
    void swap(Widget& other)
    {
        using std::swap;          // std::swap을 이 함수 안으로 끌어옴
        swap(pImpl, other.pImpl); // Widget을 맞바꾸기 위해, 각 Widget의 pImpl포인터를 맞바꾼다.
    }
private:
    WidgetImpl* pImpl;
};

namespace std {
    template<> // std::swap 템플릿의 특수화 함수
    void swap<Widget>(Widget& a, Widget& b)
    {
        a.swap(b); // swap 멤버 함수 호출
    }
}
```

## SWAP 정리
### 표준에서 제공하는 swap이 클래스 및 클래스 템플릿에 납득할만한 효율을 보이면 아무것도 하지 않는다.
- 표준 `swap`을 호출하게 될 것이다.
  
### 표준 swap의 효율성이 느리게 동작할 여지가 있다면 public swap 멤버함수를 제공해야 한다.
- 이 함수는 예외를 던져서는 안된다.
    - `swap`을 진짜 쓸모 있게 응용하는 방법들 중에 클래스가 강력한 [예외 안전성 보장](/Chapter5/Item29.md)을 제공하도록 도움을 주는 방법이 있기 때문이다.
- **멤버함수** `swap`을 제공했으면, 멤버를 호출하는 **비멤버** `swap`도 제공해야 한다.
- 새로운 클래스를 만들면 해당 클래스에 대해서 `std::swap`특수화 버전을 준비한다.

### 사용자 입장에서 swap을 호출할 때는, std::swap에 대한 using 선언 후에 네임스페이스 한정 없이 swap을 호출한다.
- `swap`을 호출하되, 네임스페이스 한정자를 붙이지 않는다.
```cpp
template<typename T>
void doSomething(T& obj1, T& obj2)
{
    using std::swap;
    swap(obj1, obj2);
}
```