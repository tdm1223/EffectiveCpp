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