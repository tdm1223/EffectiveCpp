# 항목 33. 상속된 이름을 숨기는 일은 피하자
### 변수의 유효범위(scope)
```cpp
int x; // 전역변수
void func()
{
    double x; // 지역변수
    cin >> x; // 지역변수 x에 값을 읽어 넣는다.
}
```
- 컴파일러는 자신이 처리하고 있는 지역 유효범위를 뒤져서 같은 이름을 가진 것이 있는지 알아본다.
    - 찾았다면 더 이상 탐색하지 않는다.

### 상속과 변수의 유효범위
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
- **기본 클래스**에 있는 함수들 중 `f1`, `f3`이라는 이름이 붙은 것은 모두 **파생 클래스**에 들어 있는 `f1`, `f3`에 의해 가려진다.
- `Base::f1`과 `Base::f3`은 `Derived`가 상속한것이 아니게 된다.

- **라이브러리** 혹은 **응용프로그램** 프레임 워크를 이용하여 **파생 클래스**를 만들 때 멀리 떨어져 있는 기본 클래스로부터 오버로드 버전을 상속시키는 경우를 막기위한 용도이다.

### 이름 가리기를 무시하는 방법
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
- 가려진 이름은 `using` 선언을 통해 꺼낼 수 있다
- 기본 클래스로부터 상속을 받으려고 하는데, 오버로드된 함수가 그 클래스에 들어 있고 이 함수들 중 몇개만 **오버라이드** 하고 싶다면, 각 이름에 대해 `using` 선언을 붙여주어야 한다.

### 전달 함수(forwarding function)
- 기본 클래스가 가진 함수를 전부 상속했으면 하는 것이 아닌 경우도 존재한다.
    - `is-a` 관계가 깨지기 때문에 `public` 상속은 함께 놓고 생각하지 말아야 한다.
- `using` 선언을 하면 해당되는 것들이 **모두 파생 클래스**로 내려간다.
    - **전달 함수**를 만들어 놓는것을 통해 해결할 수 있다.
```cpp
class Derived: private Base{
public:
    virtual void f1() { Base::f1(); } // 전달 함수. 암시적으로 인라인 함수가 된다.
}
```