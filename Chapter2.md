## 컴파일러가 은근슬쩍 만드는 함수
1. 기본 생성자
2. 복사 생성자
3. 복사 대입 연산자
4. 소멸자

- 복사 대입 연산자의 자동 생성은 적법하고 이치에 닿아야만 한다.
```cpp
template<class T>
class NameObject{
public:
    NameObject(std::string& name, const T& value);

private:
    std::string& nameValue; // 참조자
    const T objectValue; // 상수
};

std::string one("one");
std::string two("two");
NameObject<int> p(one, 2);
NameObject<int> q(two, 3);
p=q; // 에러!
```

## 컴파일러가 만들어낸 함수가 필요없다면 사용을 금해버리자
- 세상에 하나밖에 없는 객체가 있다. 해당 객체는 사본을 만드는것 자체가 이치에 맞지 않는다.
- 하지만 컴파일러가 복사 생성자 및 복사 대입 연산자를 은근슬쩍 만들기 때문에 복사의 가능성이 존재한다.

1. 복사 생성자 및 복사 대입 연산자를 `private` 멤버로 선언한다.
    - 여전히 그 클래스의 멤버 함수 및 `friend` 함수가 호출할 수 있는 위험이 존재한다.
2. 해당 함수들을 클래스에 **선언**만 한다.
    - 선언만 해둔다면 객체의 복사를 시도하려고 할때 컴파일러가 못하도록 막을것이다.

```cpp
class test
{
public:
    test();
private:
    test(const test&);
    test& operator=(const test&);
};
int main()
{
    test a;
    test b;
    a = b; // 에러
    a(b);  // 에러
    return 0;
}

```

3. 위에서 선언만한 복사 생성자와 복사 대입연산자를 해당 클래스에 넣지 말고 별도의 **기본 클래스**에 넣고 해당 **클래스로부터 파생**시킨다.
    - `class test: private Uncopyable` : 복사 생성자, 복사 대입연산자도 선언되지 않는다. 
    - 굳이 `public` 상속을 받을 필요가 없다.
    - 다중 상속의 문제가 발생할 수 있는데, 다중 상속 시에는 공백 기본 클래스 최적화가 돌아가지 못할때가 있다. 하지만 무시해도 아무 상관없다.

## 다형성을 가진 기본클래스에서는 소멸자를 반드시 가상 소멸자로 선언하기
```cpp
class TimeKeeper{
public:
    TimeKeeper();
    ~TimeKeeper();    
}

class AtomicClock: public TimeKeeper { ... };
class WaterClock: public TimeKeeper { ... };
class WristClock: public TimeKeeper { ... };
```

- 위와 같은 구조에서 어떤 객체에 대한 포인터를 얻기위한 아래와 같은 `팩토리 함수`를 만들어 놓는다.
```cpp
TimeKeeper *getTimeKeeper();
```
- 반환되는 객체는 **힙**에 있게 되므로, 적절하게 객체를 삭제해야 한다.
- 포인터를 통해 날아오는 **서브 클래스 객체**는 **기본 클래스 포인터**를 삭제할때 기본클래스 객체의 소멸자만 호출되고 서브클래스 객체의 소멸자는 실행되지 않는다. (부분소멸)
- 부분소멸이 일어났으니 당연히 자원의 누수가 발생할것이다. 이 때 기본클래스의 소멸자에 `virtual`키워드를 붙여줌으로써 해결할 수 있다.
- `virtual`키워드를 붙여주면 서브 클래스까지 모두 소멸자가 호출된다!
- 가상 함수를 하나라도 가진 클래스는 가상 소멸자를 가져야 하는게 대부분 맞다.
- 객체가 가상 소멸자를 가지고 있지 않다면 **기본 클래스로 사용되지 않을것**이다라고 생각해도 된다.
- 가상 소멸자를 선언하는 것은 그 클래스에 가상 함수가 하나라도 들어 있는 경우로 한정하는것이 좋다.
- STL 컨테이너 타입(vector, list, set, unordered_map등)은 가상 소멸자가 없다!

### 순수 가상 소멸자
- 클래스가 추상 클래스(인스턴스를 못 만드는 클래스)였으면 좋겠는데 넣을 만한 순수 가상 함수가 없을때가 있다.
- 순수 가상 소멸자를 선언하는것을 통해 해당 클래스를 추상 클래스로 만들면된다.
- 주의해야 할 점은 순수 가상 소멸자의 정의를 두지 않으면 안된다는 것이다.

## 예외가 소멸자를 떠나지 못하도록 붙들어 놓기
- **소멸자**에서 예외가 나가도록 내버려두면 안된다.
- **소멸자**가 예외를 발생하지 않도록 해야 한다.
1. 예외가 발생하면 프로그램을 바로 끝낸다. 대개 `abort()`를 호출한다.
2. 발생한 예외를 삼켜버린다.

- 예외를 일으키면서 실패할 가능성이 있고 예외를 처리해야 할 필요가 있다면 예외는 **소멸자가 아닌 다른 함수에서 비롯된 것**이어야 한다.

## 객체 생성 및 소멸 과정 중에는 절대로 가상 함수를 호출하지 말기
```cpp
class Transaction{
public:
    Transaction()
    {
        logTransaction();
    }
    virtual void log Transaction() const = 0;
}

class BuyTransaction: public Transaction{
public:
    virtual void logTransaction() const;
}

BuyTransaction b;
```
### 객체가 생성될 때
- 위의 객체 b를 생성하는 과정에서 기본 클래스인 `Transaction`의 생성자가 먼저 호출되게 되는데 이때 `logTransaction`은 파생 클래스의 것이 아닌 기본 클래스의 것이 호출된다.
- 기본 클래스의 생성자는 파생 클래스의 생성자보다 먼저 실행된다.
- 기본 클래스의 생성자가 호출될 동안에는, 가상 함수는 절대로 파생 클래스 쪽으로 내려가지 않는다.
- 파생 클래스의 기본 클래스 부분이 생성되는 동안 그 객체의 타입은 기본 클래스이다.

### 객체가 소멸될 때
- 파생 클래스의 소멸자가 호출되고 나면 파생 클래스만의 데이터 멤버는 정의되지 않은 값으로 가정한다.
- 기본 클래스 소멸자에 진입할 당시의 객체는 더도 덜도 아닌 기본 클래스 객체가 된다.

### 해결방법
- `logTransaction`을 `Transaction` 클래스의 비가상 멤버로 바꾼다.