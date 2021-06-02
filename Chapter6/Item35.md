# 항목 35. 가상 함수 대신 쓸 것들도 생각해 두는 자세를 시시때때로 길러 두자
## 문제의 도입
- 게임 캐릭터를 설계할때 아래와 같이 설계했다고 가정한다.
```cpp
class GameCharacter{
public:
    virtual int healthValue() const;
}
```
- `healthValue`는 캐릭터의 체력치를 반환하는 함수로 **파생 클래스는 이 함수를 재정의** 할 수 있다.
- 순수 가상 함수로 선언되지 않은 것으로 보아 체력치를 계산하는 기본 알고리즘이 제공된다는 사실을 알 수 있다.
- 너무나도 당연한 설계!
    - 다른방법은 없을 것 인가?

## 비가상 인터페이스 관용구를 통한 템플릿 메서드 패턴
- 가상 함수는 반드시 `private` 멤버로 두어야 한다고 주장하는 사람들이 제안하는 설계이다.(가상 함수 은폐론)
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
    virtual int doHealthValue() const // 파생 클래스는 이 함수를 재정의 할 수 있다.
    {
        // 캐릭터 체력 계산 로직
    }
}
```
- [파생 클래스는 이제 healthValue 함수를 재정의 할 수 없다.](/Chapter6/Item36.md)
- 멤버함수의 본문이 **클래스 정의** 안에 들어가 있다. ([암시적으로 인라인함수로 선언된다](/Chapter5/Item30.md))
- 공개되지 않은 **가상 함수**를 **비가상 public 멤버 함수**로 감싸서 간접적으로 호출하는 **템플릿 메서드 패턴**의 한 형태이다.
- **비가상 함수 인터페이스 관용구**(NVI : non-virtual-interface idiom) 라고 많이 알려져 있다.
- 비가상 함수를 가상함수의 **랩퍼**라고 부른다.

## 함수 포인터로 구현한 전략 패턴
- 비가상 함수 인터페이스 관용구는 `public` 가상 함수를 대신할 수 있는 방법이다.
  - 가상함수를 사용하는 것은 여전하다.
- 캐릭터의 체력치를 계산하는 데 가상 함수를 사용하는 것은 여전하지만 체력 계산이 굳이 캐릭터의 일부일 필요는 없다.
- 각 캐릭터의 생성자에 체력치 계산용 함수의 포인터를 넘기게 만든 후 이 함수를 호출해서 실제 계산을 수행하도록 한다.
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
- **전략 패턴**의 응용예이다.
- 가상 함수를 함수 포인터 데이터 멤버로 대체한다.
- 같은 캐릭터 타입으로 만들어진 객체들도 체력치 계산 함수를 각각 다르게 가질 수 있다.
- 특정 캐릭터에 대한 체력치 계산 함수를 바꿀 수 있다.
- 체력치 계산 함수가 `GameCharacter` 클래스의 멤버 함수가 아니기 때문에, 체력치가 계산되는 대상 객체의 비공개 데이터는 이 함수로 접근할 수 없다.

### function으로 구현한 전략 패턴
- `function` 타입의 객체를 써서 기존의 **함수 포인터를 대신**하게 만들 수 있다.
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
- `GameCharacter`가 `function` 객체(일반화된 함수 포인터)를 물게 된다는 점이 다르다.

## 고전적인 전략 패턴
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
- 일반적인 **전략 패턴** 구현 방법이다.
- 한쪽 클래스 계통에 속해 있는 가상 함수를 다른 쪽 계통에 속해 있는 가상 함수로 대체한다.
- `HealthCalcFunc` 클래스 계통에 파생 클래스를 추가함으로써 기존의 체력치 계산 알고리즘을 조정/개조 할 수 있는 가능성을 열어 놓았다.