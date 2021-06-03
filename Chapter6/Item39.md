# 항목 39. private 상속은 심사숙고해서 구사하자
## private 상속
1. 클래스 사이의 상속 관계가 `private`이면 컴파일러는 일반적으로 **파생 클래스** 객체를 **기본 클래스** 객체로 변환하지 않는다.
2. 기본 클래스로부터 물려받은 멤버는 파생 클래스에서 `private` 멤버가 된다.
    - 기본 클래스에서 원래 `protected` 멤버였거나 `public` 멤버였어도 `private` 멤버가 된다.
- `private` 상속은 `is-implemented-in-terms-of`이다.
    - B 클래스로부터 `private` 상속을 통해 D 클래스를 파생시킨 것은 B 클래스에서 쓸 수 있는 기능들 몇 개를 활용할 목적으로 한 행동이다.
    - B 타입과 D 타입의 객체 사이에 **개념적 관계**가 있는것은 아니다.
- `private` 상속은 **소프트웨어 설계** 도중에는 아무런 의미도 갖지 않고 **소프트웨어 구현**에만 의미를 가진다.

## private 상속을 사용해야 하는 경우
1. 비공개 멤버를 접근할 때 
2. 가상 함수를 재정의할 경우가 있을때

## private 상속 대신 public 상속에 객체 합성 조합이 자주 쓰이는 이유
1. 파생은 가능하게 하되, 파생 클래스에서 기본 클래스의 가상 함수를 재정의 할 수 없도록 설계차원에서 막고 싶을때 유용하다.
2. 컴파일 의존성을 최소화 할 수 있다.
   - 규모가 큰 시스템을 만들 때 **구성요소 분리**는 중요해질 수 있는 요소이다.
```cpp
class Widget{
private:
    class WidgetTimer : public Timer{
    public:
        virtual void onTick() const;
    };
    WidgetTimer timer;
}
```

## 공백 클래스
- 공백 클래스를 다룰때 `private` 상속을 선호할 수 밖에 없는 경우가 존재한다.
```cpp
class Empty{};
```
- 비정적 데이터 멤버가 없는 클래스
    - 가상 함수가 없다. (vtpr이 추가되기 때문에)
    - 가상 기본 클래스가 없다. (오버헤드의 원인)
- 공백 클래스는 **개념적**으로 차지하는 **메모리 공간**이 없는게 맞다
    - `C++`에서는 독립 구조의 객체는 반드시 크기가 **0을 넘어야** 한다.

### 공백 기본 클래스 최적화 (EBO : Empty Base Optimization)
```cpp
class Empty{};

class A{
private:
    int x;
    Empty e; // 공백 클래스지만 크기가 존재한다.
};

class B : private Empty{ // 기본 클래스로 만든다.
private:
    int x;
};
```
- `c++`에서는 독립 구조의 객체는 반드시 크기가 존재하기 때문에 `sizeof(A) > sizeof(int)`현상이 발생한다.
- 이를 해결하기 위해 `Empty` 객체를 멤버로 두지 말고 **기본 클래스**로 바꿈으로써 해결할 수 있다. `sizeof(B) == sizeof(int)`
- 공백 기본 클래스 최적화는 **단일 상속**에서만 적용된다.