# 설계 및 선언

## 인터페이스 설계는 제대로 쓰기엔 쉽게, 엉터리로 쓰기엔 어렵게
### 문제를 야기할 수 있는 인터페이스
- 날짜를 나타내는 `Date` 클래스에 넣을 생성자를 설계해본다.
- 아래 `Date` 클래스는 두가지 문제를 야기할 수 있다.
```cpp
class Date{
public:
    Date(int month, int day, int year);
}
```
1. 매개변수의 전달 순서가 잘못될 여지가 있다. 
    - `month : 3, day : 30`을 `month : 30, day : 3`으로 입력할 가능성이 있다.
2. 월과 일에 해당하는 숫자가 어이없는 숫자일 수 있다.
    - `day : 30` 을 `day : 50`으로 입력할 가능성이 있다.

### 해결방법
1. 일, 월, 연을 구분하는 간단한 랩퍼 타입을 만들고 이 타입을 `Date` 생성자 안에 들 수 있다.
2. **타입에 제약을 부여**하여 타입을 통해 할 수 있는 일들을 묶는다.

### 제대로 쓰기에 괜찮은 인터페이스
- 일관성 있는 인터페이스를 제공하기 위해서는 기본 제공 타입과 쓸데없이 어긋나는 동작을 피해야 한다.
- 괜찮은 인터페이스를 만들어 주는 요인중 하나는 **일관성**이다.
    - 모든 STL 컨테이너는 크기를 반환하는 `size`란 멤버 함수를 통해 일관성을 제공한다.
- 사용자 쪽에서 뭔가를 외워야 제대로 쓸 수 있는 인터페이스는 잘못 쓰기 쉽다.
    - 언제라도 잊어버릴 수 있다.

### 교차 DLL문제
- 객체 생성 시에 어떤 동적 링크 라이브러리의 `new`를 썻는데 그 객체를 삭제할 떄는 **이전의 DLL과 다른 DLL**에 있는 `delete`를 썼을경우 발생한다.
- `new/delete`가 실행되는 DLL이 달라서 꼬이게 되면 런타임 에러가 발생한다.

### 해결방법
- `shared_ptr`을 사용하면 문제를 피할 수 있다.
    - 클래스의 기본 삭제자는 `shared_ptr`이 생성된 DLL과 동일한 DLL에서 delete를 사용하도록 만들어져 있기 때문이다.
- `shared_ptr`은 다른 DLL들 사이에 이리저리 넘겨지더라도 교차 DLL 문제를 걱정하지 않아도 된다.

## 클래스 설계는 타입 설계와 똑같이 취급하기
- C++에서 새로운 클래스를 정의하는 것은 새로운 타입을 하나 정의하는것이다.

### 클래스를 설계할때 고려해야할 질문들
1. 새로 정의한 타입의 객체 생성 및 소멸을 어떻게 이루어져야 하는가
    - 생성자 및 소멸자의 설계가 바뀐다.
    - 메모리 할당 함수를 직접 작성할 경우 위 함수의 설계에도 영향을 미친다.
2. 객체 초기화는 객체 대입과 어떻게 달라야 하는가
    - 초기화와 대입을 헷갈리지 않아야 한다.
3. 새로운 타입으로 만든 객체가 값에 의해 전달되는 경우에 어떤 의미를 줄 것인가
    - 값에 의한 전달을 구현하는 쪽은 복사 생성자이다.
4. 새로운 타입이 가질 수 있는 적법한 값에 대한 제약은 무엇으로 잡을 것인가
    - 클래스 멤버의 몇 가지 조합 값은 반드시 유효해야 한다.
    - 생성자, 대입 연산자, 쓰기 함수는 특히 중요하다.
5. 기존의 클래스 상속 계통망에 맞출 것인가
    - 이미 갖고 있는 클래스로부터 상속을 시킬때 멤버 함수가 가상인가 비가상인가의 여부가 상속에 중요하다.
6. 어떤 종류의 타입 변환을 허용할 것인가
    - 암시적 변환을 허용할것이라면 타입 변환 함수를 추가한다.
    - 명시적 변환만 허용하고 싶다면?
7. 어떤 연산자와 함수를 두어야 의미가 있을까
    - 클래스 안에 선언할 함수가 여기서 결정된다.
8. 표준 함수들 중 어떤 것을 허용하지 말 것인가
    - private로 선언해야 하는 함수들
9. 새로운 타입의 멤버에 대한 접근권한을 어느 쪽에 줄 것인가
    - public, protected, private 영역 중 어디에 둘 것인가를 결정해야 한다.
10. 선언되지 않은 인터페이스로 무엇을 둘 것인가
11. 새로 만드는 타입이 얼마나 일반적인가
    - 타입 하나를 정의하는 것이 아닐지도 모른다.
    - 동일 계열의 타입군 전체라면 클래스 템플릿을 정의해야 한다.
12. 정말로 필요한 타입인가
    - 기존 클래스에 대해 기능 몇 개가 아쉬워서 파생 클래스를 새로 뽑고 있다면, 비멤버 함수나 템플릿을 몇 개 더 정의하는 편이 낫다.

## 값에 의한 전달 보다는 상수객체 참조자에 의한 전달 방식을 사용하기
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
- 첫번째 validateStudent 함수에 Student 객체 전달시 비용은 엄청나다
    - 생성자 여섯 번(Person, Student, string x 4)
    - 소멸자 여섯 번(Person, Student, string x 4)
- 함수에 클래스를 전달할때 두번째 validateStudent 함수처럼 **상수 객체 참조자**를 넘기는것이 좋다.

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

void printNameAndDisplay(Window w)
{
    std::cout<<w.name());
    w.display(); 
}

WindowWithScrollBars wwsb;
printNameAndDisplay(wwsb);
```
- 위 경우 매개변수가 복사 손실을 당하게 된다. 
- 매개변수 w가 생성되기는 하지만 `Window` 객체로 만들어지면서 `wwsb`가 `WindowWithScrollBars` 객체의 구실을 할 수 있는 부속 정보가 모두 없어진다.
- 참조에 의한 전달 방식으로 매개변수를 넘기면 **복사손실 문제**가 없어진다.
- 참조자를 전달한다는 것은 포인터를 전달한다는 것과 일맥상통하다.

- 크기가 작으면 값에 의한 전달?
    - 크기가 작다는 객체의 복사 생성자 호출이 저비용이란 뜻이 아니다.
    - 크기가 작아도 복사하는 비용은 고비용일 수 있다. (포인터 멤버가 가리키는 대상까지 복사한다면?)
- 값에의한 전달이 저비용이라고 가정해도 괜찮은 타입
    1. 기본제공 타입
    2. STL반복자
    3. 함수 객체타입
- 위 3가지 항목 외의 타입에 대해서는 상수객체 참조자에 의한 전달을 선택하는것이 좋다.

## 함수에서 객체를 반환해야 할 경우에 참조자를 반환하려고 들지 말기
- 값에 의한 전달에 숨겨진 효율을 알고 전부다 참조에 의한 전달로 바꾸려는것은 좋지않다.
- 참조자는 존재하는 객체에 붙는 다른 이름일 뿐이다.

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
- 위 클래스에서 `operator*`가 참조자를 반환하도록 만들어졌다면, 이 함수가 반환하는 참조자는 반드시 이미 존재하는 `Rational` 객체의 참조자여야 한다.
- 반환될 객체는 어디 있는가? 참조자를 반환하려면 유리수 객체를 직접 생성해야만 한다.

### 함수 수준에서 새로운 객체 만드는 방법
1. 스택
```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
    return result;
}
```
- `result`는 **지역 객체**이다. (함수가 끝날때 소멸된다)
- 이런 지역 객체를 반환한다면?
    - 더이상의 설명은 필요없다.
    
2. 힙
```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational *result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    return *result;
}
```
- `new`로 생성한 객체는 어디서 `delete`될것인가?
    - 더이상의 설명은 필요없다.

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
## 데이터 멤버가 선언될 곳은 private 영역임을 명심하기
### 데이터 멤버가 public이면 안되는 이유
1. 문법적 일관성
    - 데이터 멤버에 대해 접근할 수 있는 경우가 함수 뿐이라면 괄호를 붙여야 하는지 말아야 하는지 고려할 필요가 없다.
2. 접근성에 대해 정교한 제어
    - 접근 불가, 일기 전용, 일기 쓰기 접근을 직접 구현할 수 있다.
3. 캡슐화
    - 데이터 멤버를 계산식으로 대체할수 있다.
    - 사용자는 절대로 클래스를 넘볼 수 없다.
- 데이터 멤버를 함수 인터페이스 뒤에 감추게 되면 구현상의 융통성을 전부 누릴 수 있다.
- 캡슐화를 진행하면 클래스의 불변속성을 항상 유지하는 데 절대로 소홀해질 수 없게 된다.
- `protected`는 `public`보다 더 많이 보호받고 있는것이 절대로 아니다.
- 캡슐화 관점에서 쓸모있는 접근 수준은 `private`(캡슐화 제공)와 `private가 아닌 나머지`(캡슐화 없음) 둘뿐이다.

## 멤버 함수보다는 비멤버 비프렌드 함수와 더 가까워지기
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
- clearEverything() 함수는 비멤버 함수로 제공할 수 있다.
```cpp
void clearBrowser(WebBrowser& wb)
{
    wb.clearCache();
    wb.clearHistroy();
    wb.removeCookies();
}
```
- 어느쪽이 더 괜찮을까?
    - 비멤버 버전이 캡슐화 정도가 높고 패키징 유연성 또한 높다.

### 주의해야 할 점
- 비프렌드 함수에만 적용된다.
- `함수는 어떤 클래스의 비멤버가 되어야 한다`라는 주장이 `그 함수는 다른 클래스의 멤버가 될 수 없다` 라는 의미가 아니다.
    - C++로는 비멤버 함수로 두고 같은 네임스페이스 안에두는것으로 구사할 수 있다.
    - 네임스페이스는 클래스와 달리 여러 개의 소스 파일에 나뉘어 흩어질 수 있기 때문이다.
