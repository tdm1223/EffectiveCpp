# 항목 37. 어떤 함수에 대해서도 상속받은 기본 매개변수 값은 절대로 재정의 하지 말자
## C++에서 상속받을 수 있는 함수의 종류
1. 가상 함수
2. 비가상 함수
    - [절대로 재정의해서는 안 되는 함수](/Chapter6/Item36.md)

## 객체의 정적 타입
- 프로그램 소스 안에 **선언문**을 통해 객체가 갖는 타입
```cpp
class Shape{
public:
    enum ShapeColor {Red, Green, Blue};
    // 모든 도형은 자기 자신을 그리는 함수를 제공해야 한다.
    virtual void draw(ShapeColor color = Red) const = 0;
};

class Rectangle: public Shape{
public:
    // 기본 매개변수가 다르다.
    virtual void draw(ShapeColor color = Green) const;
};

class Circle: public Shape{
public:
    virtual void draw(ShapeColor color) const;
};

Shape * ps;
Shape * pr = new Rectangle;
Shape * pc = new Circle;
```
- `ps`, `pr`, `pc`는 **Shape에 대한 포인터**로 선언되어 있기 때문에, 정적 타입도 모두 **Shape에 대한 포인터** 타입이다.
- 실제 가리키는 대상이 달라지는것은 하나도 없다.

## 객체의 동적 타입
- 현재 객체가 **진짜로 무엇**이냐에 따라 결정되는 타입
- **객체가 어떻게 동작할 것이냐**를 가리키는 타입
- `pc`의 동적 타입은 `Circle*`이고, `pr`의 동적 타입은 `Rectangle*`, `ps`의 동적 타입은 없다.
```cpp
ps = pc; // ps의 동적 타입은 Circle*가 된다.
pc->draw(Shape::Red); // Circle::draw(Shape::Red)를 호출한다.

ps = pr; // ps의 동적 타입은 Rectangle*가 된다.
pr->draw(Shape::Red); // Rectangle::draw(Shape::Red)를 호출한다.
pr->draw(); // Rectanble의 기본 매개변수는 Green으로 위에서 설정해 두었지만 Rectangle::draw(Shape::Red)를 호출한다.
```
- `pr`의 **동적 타입**은 `Rectangle*`이므로 호출되는 가상 함수는 `Rectangle`의 것이다.
- `pr`의 **정적 타입**은 `Shape*`이므로 호출되는 가상 함수에 쓰이는 기본 매개변수 값을 `Shape` 클래스에서 가져온다.
- **파생 클래스**에 정의된 **가상 함수**를 호출하면서 **기본 클래스**에 정의된 **기본 매개변수 값**을 사용해버릴 수 있다.
- `Shape` 및 `Rectangle` 클래스 양쪽에서 선언된 것이 한데 섞이는 이상한 함수 호출이 이루어진다.
- 해결하기 위해선 [비가상 인터페이스 관용구](/Chapter6/Item35.md)를 사용하면 된다.
    - 파생 클래스에서 재정의 할 수 있는 가상 함수를 `private` 멤버로 둔다.
    - 가상 함수를 호출하는 `public` 비가상 함수를 기본 클래스에 만들어 둔다.
    - 비가상 함수가 기본 매개변수를 지정하도록 하면 기본 매개변수도 설정할 수 있다.

## 비가상 인터페이스 관용구 사용
```cpp
class Shape{
public:
    enum ShapeColor {Red, Green, Blue};
    void draw(ShapeColor color = Red) const // draw는 비가상 함수가 된다.
    {
        doDraw(color); // 가상 함수를 호출한다.
    }
private:
    virtual void doDraw(ShapeColor color) const = 0; // 진짜 작업이 이루어 지는 함수
};

class Rectangle: public Shape{
public:
private:
    virtual void doDraw(ShapeColor color) const; // 기본 매개변수 값이 없다.
};
```
- [비가상 함수는 파생 클래스에서 오버라이드 하면 안 되기 때문에](/Chapter6/Item36.md), 위처럼 설계하면 `draw` 함수의 `color` 매개변수에 대한 **기본값을 고정** 시킬 수 있다.