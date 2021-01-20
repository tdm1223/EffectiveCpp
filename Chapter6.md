# 상속과 객체 지향 설계

## public 상속 모형은 반드시 is-a를 따르도록 만들기
- 파생 클래스 타입으로 만들어진 모든 객체는 기본 클래스 타입의 객체이지만 **반대는 되지 않는다**.
    - 기본 클래스는 파생 클래스보다 더 일반적인 개념을 나타낸다.
    - 파생 클래스는 기본 클래스보다 더 특수한 개념을 나타낸다.
- `public` 상속은 기본 클래스 객체가 가진 **모든 것들**이 파생 클래스 객체에도 그대로 적용된다고 단정한다.

## 상속된 이름을 숨기는 일은 피하기
```cpp
class Base{
private:
    int x;
public:
    virtual void f1() = 0;
    virtual void f1(int);
    virtual void f2();
    void f3();
    void f3(double);
};

class Derived: public Base{
public:
    virtual void f1();
    void f3();
    void f4();
}

Derived d;
int x;

d.f1(); // Derived::f1을 호출
d.f1(x); // 에러 Derived::f1이 Base::f1을 가림

d.f2(); // Base::f2를 호출

d.f3(); // Derived::f3을 호출
d.f3(x); // 에러 Derived::f3이 Base::f3을 가림
```
- 기본 클래스에 있는 함수들 중 f1, f3이라는 이름이 붙은 것은 모두 파생클래스에 들어 있는 f1, f3에 의해 가려진다.
- `Base::f1`과 `Base::f3`은 `Derived`가 상속한것이 아니게 된다.

- **라이브러리** 혹은 **응용프로그램** 프레임 워크를 이용하여 **파생 클래스**를 만들 때 멀리 떨어져 있는 기본 클래스로부터 오버로드 버전을 상속시키는 경우를 막기위한 용도이다.
- 가려진 이름은 `using` 선언을 써서 끄집어 낼 수 있다!
```cpp
class Derived: public Base{
public:
    using Base::f1;
    using Base::f3;

    virtual void f1();
    void f3();
    void f4();
}
```
- 기본 클래스로부터 상속을 받으려고 하는데, 오버로드된 함수가 그 클래스에 들어 있고 이 함수들 중 몇개만 오버라이드 하고 싶다면, 각 이름에 대해 `using` 선언을 붙여주어야 한다.

### 전달 함수(forwarding function)
- 기본 클래스가 가진 함수를 전부 상속했으면 하는 것이 아닌 경우도 존재한다.
    - `is-a` 관계가 깨지기 때문에 public 상속은 함께 놓고 생각하지 말아야 한다.
- `using` 선언을 내리면 그 이름에 해당되는 것들이 **모두 파생 클래스**로 내려간다.
    - 전달 함수를 만들어 놓는것을 통해 해결할 수 있다.
```cpp
class Derived: private Base{
public:
    virtual void f1() { Base::f1(); } // 전달 함수. 암시적으로 인라인 함수가 된다.
}
```

## 인터페이스 상속과 구현 상속의 차이를 제대로 파악하고 구별하기
### public 상속의 두 가지 개념
1. 인터페이스 상속
2. 구현 상속

```cpp
class Shape{
public:
    virtual void draw() const = 0; // 순수 가상 함수
    virtual void error(const std::string& msg); // 단순 가상 함수
    int objectID() const; // 비가상 함수
};

class Rectangle: public Shape{};
class Ellipse: public Shape{};

Shape a; // 당연한 에러
```
- `Shape`는 순수 가상 함수인 draw 함수 때문에 인스턴스를 만들 수 없는 **추상 클래스**이다.
- `Shape`의 파생 클래스(Rectangle, Ellipse)만 인스턴스를 만들 수 있다.

### 각 함수에 대한 설명
1. draw 함수(순수 가상 함수)
    - 순수 가상 함수를 물려받은 클래스는 해당 순수 가상 함수를 **다시 선언**해야 한다.
    - 순수 가상 함수는 추상 클래스 안에서 정의를 갖지 않는다.
    - 순수 가상 함수를 선언하는 목적은 파생 클래스에게 함수의 **인터페이스**만을 물려주려는 것이다.
    - 순수 가상 함수에도 **정의를 제공**할 수 있다. 
        - `virtual void draw() const = 0 { std::cout<<"Shape Draw"<<std::endl; }`
        - 클래스 이름을 **한정자**로 붙여주어야만 동작한다. `Shape::draw()`

2. error 함수(단순 가상 함수)
    - 단순 가상 함수를 선언하는 목적은 **파생 클래스**로 하여금 함수의 인터페이스뿐만 아니라 그 함수의 기본 구현도 물려받게 하자는 것이다.
    - 단순 가상 함수는 프로그래머가 지원해야 하지만 새로 만들 필요가 없다면 기본 클래스에 있는 버전을 사용할 수 있다.
    - 특별히 처리해야 한다면 오버라이드를 통해 제공하면 된다. 

3. objectID 함수(비가상 함수)
    - 비가상 함수를 선언하는 목적은 파생 클래스가 함수 **인터페이스**와 더불어 그 함수의 **필수적인 구현**을 물려받게 하는 것이다.
    - `Shape 및 Shape에서 파생된 모든 객체는 객체의 식별자를 내어주는 같은 함수를 갖게될것이다.`라고 생각하면 된다.
    - 비가상 함수는 클래스 파생에 상관없는 **불변동작**과 같기 때문에, 파생 클래스에서 재정의할 수 있는 것이 절대로 아니다.

### 클래스 설계시 주의할점
- 순수 가상 함수, 단순 가상 함수, 비가상 함수의 선언문이 가진 차이점 덕분에 파생 클래스가 물려받았으면 하는 것들을 정밀하게 지정할 수 있다.
- 판단에 따라 인터페이스만 상속시켜도 되고, 인터페이스와 기본 구현을 함께 상속시킬 수도 있고, 인터페이스와 필수 구현을 상속 시킬 수 있다.
- 어떤 클래스에 멤버 함수를 선언해 넣을때 많이 설계해 보지 못했다면 아래와 같은 실수를 할 수 있다.
1. 모든 멤버를 **비가상 함수**로 선언하는 것
    - 파생 클래스를 만들더라도 기본 클래스의 동작을 특별하게 만들만한 여지가 없어지게 된다.
    - 비가상 소멸자가 문제가 될 수 있다.
2. 모든 멤버를 **가상 함수**로 선언하는 것
    - 파생 클래스에서 재정의가 안 되어야 하는 함수도 있을것이다. 이런 함수는 반드시 비가상 함수로 만들어서 입장을 밝혀야 한다.
    - 인터페이스 클래스처럼 맞는 경우도 존재한다.

## 가상 함수 대신 쓸 것들도 생각해 두는 자세를 길러두기
- 게임 캐릭터를 설계할때 아래와 같이 설계했다고 가정한다.
```cpp
class GameCharacter{
public:
    virtual int healthValue() const;
}
```
- `healthValue`는 캐릭터의 체력치를 반환하는 함수로 파생 클래스는 이 함수를 재정의 할 수 있다.
- 너무나도 당연한 설계!
- 다른방법은 없을것인가?

### 비가상 인터페이스 관용구를 통한 템플릿 메서드 패턴
- 가상 함수는 반드시 private 멤버로 두어야 한다고 주장하는 사람들이 제안하는 설계이다.
```cpp
class GameCharacter{
public:
    int healthValue() const
    {
        // 사전 동작

        int retval = doHealthValue(); // 실제 동작

        // 사후 동작

        return retVal;
    }
private:
    virtual int doHealthValue() const
    {
        // 캐릭터 체력 계산 로직
    }
}
```
- 멤버함수의 본문이 클래스 정의 안에 들어가 있다. (암시적으로 인라인함수로 선언된다)
- 공개되지 않은 **가상 함수**를 **비가상 public 멤버 함수**로 감싸서 간접적으로 호출하는 템플릿 메서드 패턴의 한 형태이다.
- **비가상 함수 인터페이스 관용구**라고 많이 알려져 있다.
- 비가상 함수를 가상함수의 랩퍼라고 부른다.

### 함수 포인터로 구현한 전략 패턴
- 비가상 함수 인터페이스 관용구는 `public` 가상 함수를 대신할 수 있는 괜찮은 방법이다.
- 캐릭터의 체력치를 계산하는 데 가상 함수를 사용하는 것은 여전하고, 체력치 계산이 굳이 캐릭터의 일부일 필요는 없다.
- 각 캐릭터의 생성자에 체력치 계산용 함수의 포인터를 넘기게 만들고, 이 함수를 호출해서 실제 계산을 수행하도록 한다.
```cpp
class GameCharacter; // 전방 선언

int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter{
public:
    typedef int (*HealthCalcFunc) (const GameCharacter&);

    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf) { }
    int health Value() const
    {
        return healthFunc(*this);
    }
private:
    HealthCalcFunc healthFunc;
```
- 전략 패턴의 응용예이다.
- 가상 함수를 함수 포인터 데이터 멤버로 대체한다.
- 같은 캐릭터 타입으로 만들어진 객체들도 체력치 계산 함수를 각각 다르게 가질 수 있다.
- 특정 캐릭터에 대한 체력치 계산 함수를 바꿀 수 있다.
- 체력치 계산 함수가 `GameCharacter` 클래스의 멤버 함수가 아니기 때문에, 체력치가 계산되는 대상 객체의 비공개 데이터는 이 함수로 접근할 수 없다.

### function으로 구현한 전략 패턴
- `function` 타입의 객체를 써서 기존의 함수 포인터를 대신하게 만들 수 있다.
- `function` 계열의 객체는 함수 포인터를 가질 수 있고, 이들 개체는 주어진 시점에서 예상되는 시그니처와 호환되는 시그니처를 갖고 있다.

```cpp
class GameCharacter;

int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter {
public:
    typedef std::function<int(const GameCharacter&)> HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf) {}
    int healthValue() const
    {
        return healthFunc(*this);
    }
private:
    HealthCalcFunc healthFunc;
};
```
- 가상 함수를 `function` 데이터 멤버로 대체한다.
- 함수 포인터로 구현한 것과 큰 차이는 없다.
- `GameCharacter`가 이제는 `function` 객체(좀더 일반화된 함수 포인터)를 물게 된다는 점이 다르다.

### 고전적인 전략 패턴
```cpp
class GameCharacter;

// 최상위 클래스
class HealthCalcFunc {
public:
    virtual int calc(const GameCharacter& gc) const;
};

HealthCalcFunc defaultHealthCalc;

// 최상위 클래스
class GameCharacter {
public:
    explicit GameCharacter(HealthCalcFunc *phcf = &defaultHealthCalc) : pHealthCalc(phcf) {}
    int healthValue() const
    {
        return pHealthCalc->calc(*this);
    }
private:
    HealthCalcFunc * pHealthCalc; // 최상위 클래스인 HealthCalcFunc의 포인터를포함하고 있음
};
```
- 표준적인 전략 패턴 구현 방법이다.
- 한쪽 클래스 계통에 속해 있는 가상 함수를 다른 쪽 계통에 속해 있는 가상 함수로 대체한다.
- `HealthCalcFunc` 클래스 계통에 파생 클래스를 추가함으로써 기존의 체력치 계산 알고리즘을 조정/개조 할 수 있는 가능성을 열어 놓았다.