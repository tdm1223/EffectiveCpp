# 설계 및 선언

## 항목 18, 인터페이스 설계는 제대로 쓰기엔 쉽게, 엉터리로 쓰기엔 어렵게 하자
- 인터페이스를 사용하는데 결과 코드가 사용자가 생각한 대로 동작하지 않는다면 그 코드는 컴파일 되지 않아야 맞다.

### 문제를 야기할 수 있는 인터페이스
- 아래 날짜를 나타내는 `Date` 클래스는 두가지 문제를 야기할 수 있다.
```cpp
class Date{
public:
    Date(int month, int day, int year);
}
```
1. 매개변수의 전달 순서가 잘못될 여지가 있다. 
    - `month : 3, day : 30`을 `month : 30, day : 3`으로 입력할 가능성이 있다.
2. 매개변수에 비정상적인 숫자가 들어갈 여지가 있다.
    - `day : 30` 을 `day : 50`으로 입력할 가능성이 있다.

### 해결방법
1. 일, 월, 연을 구분하는 간단한 **랩퍼 타입**을 만들고 이 타입을 `Date` 생성자 안에 들 수 있다.
2. **타입에 제약을 부여**하여 타입을 통해 할 수 있는 일들을 묶는다.
```cpp
struct Day{
    explicit Day(int d) : val(d){}
    int val;
};
struct Month{
    explicit Month(int d) : val(d){}
    int val;
};
struct Year{
    explicit Year(int d) : val(d){}
    int val;
};
class Date{
public:
    Date(const Month& m, const Day& d, const Year& y);
};
```

### 사용하기에 좋은 인터페이스
- 일관성 있는 인터페이스를 제공하기 위해서는 **기본 제공 타입**과 어긋나는 동작을 피해야 한다.
- 좋은 인터페이스를 만들어 주는 요인중 하나는 **일관성**이다.
    - 모든 STL 컨테이너는 크기를 반환하는 `size`란 멤버 함수를 통해 일관성을 제공한다.
- 사용자 쪽에서 암기를 통해 제대로 쓸 수 있는 인터페이스는 잘못 쓰기 쉽다.
    - 언제라도 잊어버릴 수 있다.

### 교차 DLL문제
- 객체 생성 시에 어떤 동적 링크 라이브러리의 `new`를 썻는데 그 객체를 삭제할 떄는 **이전의 DLL과 다른 DLL**에 있는 `delete`를 썼을경우 발생한다.
- `new/delete`가 실행되는 `DLL`이 달라서 꼬이게 되면 런타임 에러가 발생한다.

### 해결방법
- `shared_ptr`을 사용하면 문제를 피할 수 있다.
    - 클래스의 기본 삭제자는 `shared_ptr`이 생성된 DLL과 동일한 DLL에서 delete를 사용하도록 만들어져 있기 때문이다.
- `shared_ptr`은 다른 DLL들 사이에 이리저리 넘겨지더라도 교차 DLL 문제를 걱정하지 않아도 된다.

## 항목 19. 클래스 설계는 타입 설계와 똑같이 취급하자
- C++에서 **새로운 클래스**를 정의하는 것은 **새로운 타입**을 하나 정의하는것이다.

### 클래스를 설계할때 고려해야할 질문들
1. 새로 정의한 타입의 객체 생성 및 소멸은 어떻게 이루어져야 하는가
    - **생성자** 및 **소멸자**의 설계가 바뀐다.
    - **메모리 할당 함수**를 직접 작성할 경우 위 함수의 설계에도 영향을 미친다.
2. **객체 초기화**는 **객체 대입**과 어떻게 달라야 하는가
    - 생성자와 대입 연산자의 동작과 둘 사이의 차이점을 결정 짓는 요소이다.
    - 초기화와 대입을 헷갈리지 않아야 한다.
3. 새로운 타입으로 만든 객체가 **값에 의해 전달**되는 경우에 어떤 의미를 줄 것인가
    - **값에 의한 전달**을 구현하는 쪽은 **복사 생성자**이다.
4. 새로운 타입이 가질 수 있는 값에 대한 제약은 무엇으로 잡을 것인가
    - 클래스 멤버의 몇 가지 조합 값은 반드시 유효해야 한다. (클래스의 불변속성)
    - 생성자, 대입 연산자, 쓰기 함수는 특히 중요하다.
5. 기존의 클래스 상속 계통망에 맞출 것인가
    - 이미 갖고 있는 클래스로부터 상속을 시킬때 멤버 함수가 **가상**인가 **비가상**인가의 여부가 상속에 중요하다.
6. 어떤 종류의 타입 변환을 허용할 것인가
    - **암시적 변환**을 허용하고 싶다면?
        - 타입 변환 함수를 만든다.
        - 인자 한 개로 호출될 수 있는 비명시호출 생성자를 만든다.
    - **명시적 변환**만 허용하고 싶다면?
        - 해당 변환을 맡는 별도 이름의 함수를 만들되 타입 변환 연산자 혹은 비명시호출 생성자는 만들지 말아야 한다.
7. 어떤 연산자와 함수를 두어야 의미가 있을까
    - 클래스 안에 선언할 함수가 여기서 결정된다.
        - 멤버 함수로 적당한것, 그렇지 않은것...
8. 표준 함수들 중 어떤 것을 허용하지 말 것인가
    - `private`로 선언해야 하는 함수들
9. 새로운 타입의 멤버에 대한 **접근권한**을 어느 쪽에 줄 것인가
    - `public`, `protected`, `private` 중 어느것으로 선언할지 결정해야 한다.
10. 선언되지 않은 인터페이스로 무엇을 둘 것인가
    - 만드는 타입의 수행 성능, 예외 안전성, 자원사용(잠금 및 동적메모리)이 보장되어야 한다.
11. 새로 만드는 타입이 얼마나 일반적인가
    - 타입 하나를 정의하는 것이 아닐지도 모른다.
    - 동일 계열의 타입군 전체라면 **클래스 템플릿**을 정의해야 한다.
12. 정말로 필요한 타입인가
    - 기존 클래스에 대해 기능 일부가 아쉬워서 파생 클래스를 새로 만들고 있다면?
        - **비멤버 함수**나 **템플릿**을 몇 개 더 정의하는 편이 낫다.

## 항목 20. 값에 의한 전달 보다는 상수객체 참조자에 의한 전달 방식을 택하는 편이 대개 낫다
### 값을 전달한다면?
```cpp
class Person{
public:
    std::string name;
    std::string address;
};

class Student : public Person{
public:
    std::string schoolName;
    std::string schoolAddress;
};

bool validateStudent(Student s);
bool validateStudent(const Student& s);
```
- 첫번째 `validateStudent` 함수에 `Student` 객체 전달시 비용은 엄청나다
    - 생성자 **여섯 번**(Person, Student, string x 4)
    - 소멸자 **여섯 번**(Person, Student, string x 4)
- 함수에 클래스를 전달할때 두번째 `validateStudent` 함수처럼 **상수 객체 참조자**를 넘기는것이 좋다.

### 복사 손실 문제
```cpp
class Window{
public:
    std::string name() const;
    virtual void display() const;
};

class WindowWithScrollBars : public Window{
public:
    virtual void display() const;
};

void printNameAndDisplay(Window w) // 복사 손실을 당하는 매개변수
{
    std::cout<<w.name());
    w.display(); 
}

WindowWithScrollBars wwsb;
printNameAndDisplay(wwsb);
```
- **매개변수**가 **복사 손실**을 당하게 된다. 
- 매개변수 `w`가 생성되기는 하지만 `Window` 객체로 만들어지면서 `wwsb`가 `WindowWithScrollBars` 객체의 구실을 할 수 있는 부속 정보가 모두 없어진다.
- 참조에 의한 전달 방식으로 매개변수를 넘기면 **복사손실 문제**가 없어진다.
    - `void printNameAndDisplay(const Window& w);`
- **참조자를 전달**한다는 것은 **포인터를 전달**한다는 것과 일맥상통하다.
- 크기가 작으면 값에 의한 전달?
    - 크기가 작다는 객체의 복사 생성자 호출이 저비용이란 뜻이 아니다.
    - 크기가 작아도 **복사하는 비용**은 **고비용**일 수 있다. (포인터 멤버가 가리키는 대상까지 복사한다면?)
- 값에 의한 전달이 저비용이라고 가정해도 괜찮은 타입
    1. 기본제공 타입
    2. STL반복자
    3. 함수 객체타입
- 위 3가지 항목 외의 타입에 대해서는 상수객체 참조자에 의한 전달을 선택하는것이 좋다.

## 항목 21. 함수에서 객체를 반환해야 할 경우에 참조자를 반환하려고 들지 말자
- 값에 의한 전달에 효율을 알았다고 모두 참조에 의한 전달로 바꾸려는것은 좋지않다.
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
    - 이 함수가 반환하는 참조자는 반드시 이미 **존재하는** `Rational` 객체의 참조자여야 한다.
- 반환될 객체는 어디 있는가? 
    - 참조자를 반환하려면 유리수 객체를 **직접 생성**해야만 한다.

### 함수 수준에서 새로운 객체 만드는 방법
- 함수 수준에서 새로운 객체를 만드는 방법은 스택에 만드는 것, 힙에 만드는 것 두가지 뿐이다.

1. 스택
```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
    return result;
}
```
- `result`는 **지역 객체**이다. (함수가 끝날때 **소멸**된다)
- 이런 지역 객체를 반환한다면?
    - 반환되는것은 Rational 객체였던 소멸자가 호출된 바이트 덩어리가 반환될 것이다.
    
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
- 스레드 안정성의 문제
- `operator==`연산자 오버로딩시 항상 같은 값을 반환할 수 있다(정적 객체)

### 정적 데이터를 배열로 쓴다면?
- 배열의 크기를 먼저 정해야 한다.
    - 적당하게 해야한다. 작으면 반환값을 저장할 공간이 부족하고 크다면 수행 성능이 떨어진다. (불가능)

### BEST 코드
```cpp
inline const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```
- 새로운 객체를 반환하게 만든다.
- 객체 생성, 소멸의 비용이 들어가지만 올바른 동작에 지불되는 작은 비용일 뿐이다.

## 항목 22. 데이터 멤버가 선언될 곳은 private 영역임을 명심하자
### 데이터 멤버가 public이면 안되는 이유
1. 문법적 일관성
    - 데이터 멤버에 대해 접근할 수 있는 경우가 함수 뿐이라면 괄호를 붙여야 하는지 말아야 하는지 고려할 필요가 없다.
2. 접근성에 대해 정교한 제어
    - 접근 불가, 읽기 전용, 읽기 쓰기 접근을 직접 구현할 수 있다.
3. **캡슐화**
    - 데이터 멤버를 계산식으로 대체할수 있다.
    - 사용자는 절대로 클래스를 넘볼 수 없다.
- 데이터 멤버를 함수 인터페이스 뒤에 감추게 되면 구현상의 융통성을 전부 누릴 수 있다.
- 캡슐화를 진행하면 클래스의 불변속성을 유지하게 된다.
- `protected`는 `public`보다 더 많이 보호받고 있는것이 절대로 아니다.
- **캡슐화 관점**에서 쓸모있는 접근 수준은 `private`(캡슐화 제공)와 `private가 아닌 나머지`(캡슐화 없음) 둘뿐이다.

## 항목 23. 멤버 함수보다는 비멤버 비프렌드 함수와 더 가까워지자
### 캡슐화
- 캡슐화하면 외부에서 볼 수 없게 된다.
- 캡슐화 하는 것이 늘어나면 밖에서 볼 수 있는 것들이 줄어든다.
- 밖에서 볼 수 있는 것이 줄어들면 그것들을 바꿀 때 필요한 유연성이 커진다.

### 웹브라우저를 나타내는 클래스가 있다고 가정
```cpp
class WebBrowser{
public:
    void clearCache();
    void clearHistory();
    void removeCookies();
    void clearEverything(); // 위 3개를 한번에 호출해 주는 함수
}
```
- `clearEverything` 함수는 멤버 함수가 아닌 **비멤버 함수**로 제공할 수 있다.
```cpp
void clearBrowser(WebBrowser& wb)
{
    wb.clearCache();
    wb.clearHistroy();
    wb.removeCookies();
}
```
- 어느쪽이 더 괜찮을까?
    - **비멤버 함수**가 캡슐화 정도가 높고 **패키징 유연성** 또한 높다.
    - 컴파일 의존도도 낮추고 `WebBrowser`클래스의 확장성도 높일 수 있다.

### 주의해야 할 점
1. 비프렌드 함수에만 적용된다.
    - 프렌드 함수는 `private` 멤버에 대한 접근 권한이 해당 클래스의 멤버 함수가 가진 접근 권한과 같기 때문에, 캡슐화에 대한 영향도 같다.
2. `함수는 어떤 클래스의 비멤버가 되어야 한다`라는 주장이 `그 함수는 다른 클래스의 멤버가 될 수 없다` 라는 의미가 아니다.
    - 비멤버 함수로 두고 **같은 네임스페이스** 안에 두는것으로 구사할 수 있다.
    - 네임스페이스는 클래스와 달리 여러 개의 소스 파일에 나뉘어 흩어질 수 있기 때문이다.

## 항목 24. 타입 변환이 모든 매개변수에 대해 적용되어야 한다면 비멤버 함수를 선언하자
### 유리수를 나타내는 클래스
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

### 곱셈 지원
```cpp
const Rational operator* (const Rational& rhs) const; // 멤버함수로 추가!
result = oneHalf * 2; // 성공 result = oneHalf.operator*(2)
result = 2 * oneHalf; // 에러 result = 2.operator*(onehalf)
```
- **혼합형 수치연산**을 지원하지 못하는 문제가 생기게 된다.

### 알수 있는 사실
- **암시적 타입 변환**에 대해 매개변수가 먹혀들려면 **매개변수 리스트**에 들어 있어야만 한다.
- `operator*`를 **비멤버 함수**로 만들어서, 컴파일러 쪽에서 **모든 인자**에 대해 **암시적 타입 변환**을 수행하도록 한다.
```cpp
const Rational operator*(const Rational* lhs, const Rational* rhs)
{
    return Rational (lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}
```
- 멤버 함수의 반대는 프렌드 함수가 아니라 비멤버 함수이다.

## 항목 25. 예외를 던지지 않는 swap에 대한 지원
```cpp
namespace std{
    template<typename T>
    void swap(T& a, T& b)
    {
        T temp(a);
        a = b;
        b = temp;
    }
}
```
- 표준에서 기본적으로 제공하는 `swap`은 **복사**만 제대로 지원하는 타입이라면 어떤 타입의 객체이든 맞바꾸기 동작을 수행해 준다.
- 한 번 호출에 복사가 **세 번** 일어난다.

1. 표준에서 제공하는 swap이 클래스 및 클래스 템플릿에 납득할만한 효율을 보이면 아무것도 하지 않는다.
    - 표준 `swap`을 호출하게 될 것이다.
2. 표준 swap의 효율성이 느리게 동작할 여지가 있다면 `public swap` **멤버함수**를 제공해야 한다.
    - 이 함수는 예외를 던져서는 안된다.
        - `swap`을 진짜 쓸모 있게 응용하는 방법들 중에 클래스가 강력한 예외 안전성 보장을 제공하도록 도움을 주는 방벙이 있기 때문이다.
    - **멤버함수** `swap`을 제공했으면, 멤버를 호출하는 **비멤버** `swap`도 제공해야 한다.
    - 새로운 클래스를 만들면 해당 클래스에 대해서 `std::swap`특수화 버전을 준비한다.
3. 사용자 입장에서 `swap`을 호출할 때는, `std::swap`에 대한 `using` 선언 후에 네임스페이스 한정 없이 `swap`을 호출한다.
```cpp
template<typename T>
void doSomething(T& obj1, T& obj2)
{
    using std::swap;
    swap(obj1, obj2);
}
```