# 항목 9. 객체 생성 및 소멸 과정 중에는 절대로 가상 함수를 호출하지 말자
- 아래와 같이 주식 거래를 본떠 만든 클래스 계통 구조가 있다고 가정한다.
```cpp
class Transaction{ // 모든 거래에 대한 기본 클래스
public:
    Transaction()
    {
        logTransaction();
    }
    virtual void log Transaction() const = 0; // 로그 타입에 따라 달라지는 로그 기록
}

class BuyTransaction: public Transaction{
public:
    virtual void logTransaction() const;
}

BuyTransaction b;
```
## 객체가 생성될 때
- 객체 b를 생성하는 과정에서 기본 클래스인 `Transaction`의 생성자가 먼저 호출되게 되는데 이때 `logTransaction`은 파생 클래스의 것이 아닌 기본 클래스의 것이 호출된다.
- **기본 클래스 생성자**는 **파생 클래스 생성자**보다 먼저 실행된다.
- **기본 클래스 생성자**가 호출될 동안에는, 가상 함수는 절대로 **파생 클래스** 쪽으로 내려가지 않는다.
- **파생 클래스**의 **기본 클래스** 부분이 생성되는 동안 그 객체의 타입은 **기본 클래스**이다.

## 객체가 소멸될 때
- **파생 클래스 소멸자**가 호출되고 나면 파생 클래스만의 데이터 멤버는 정의되지 않은 값으로 가정한다.
- **기본 클래스 소멸자**에 진입할 당시의 객체는 기본 클래스 객체가 된다.

## 해결방법
- `logTransaction`을 `Transaction` 클래스의 **비가상 멤버**로 바꾼다.
- 파생 클래스의 생성자들이 필요한 로그 정보를 `Transaction`의 **생성자**로 넘겨야 한다는 규칙을 만든다.

```cpp
class Transaction {
public:
    explicit Transaction(const string& logInfo);
    Transaction(const string& logInfo) {
        logTransaction(logInfo); // 비가상 함수 호출
    }
    void logTransaction(const string& logInfo) const;
};

class BuyTransaction : public Transaction {
public:
    BuyTransaction() : Transaction(createLogString(params)) {} // 로그 정보를 기본 클래스 생성자로 넘긴다.
private:
    static string createLogString(params);
};
```
- 기본 클래스 부분이 생성될 때는 가상 함수를 호출한다 해도 기본 클래스의 울타리를 넘어갈 수 없다.
  - 필요한 초기화 정보를 파생 클래스 쪽에서 기본 클래스 생성자로 올려주도록 만드는 방법이다.