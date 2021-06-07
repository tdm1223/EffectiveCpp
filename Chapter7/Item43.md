# 항목 43. 템플릿으로 만들어진 기본 클래스 안의 이름에 접근하는 방법을 알아두자
## 여러 회사에 메시지를 전송할 수 있는 응용프로그램을 만든다고 가정
- 전송용 메시지는 암호화, 비암호화 전송 2가지 방식이 존재한다.
- 어떤 메시지가 어떤 회사로 전송될지를 컴파일 도중에 결정할 수 있는 충분한 정보가 있다면 **템플릿 기반**의 방법을 쓸 수 있다.

```cpp
class CompanyA{
public:
    void sendCleartext(const std::string& msg);
};

class CompanyB{
public:
    void sendCleartext(const std::string& msg);
};

class CompanyZ{
public:
    void sendEncrypted(const std::string& msg);
};

// MsgSender 템플릿의 완전 특수화 버전
// sendClear함수가 존재하지 않는다.
template<>
class MsgSender<CompanyZ>{
public:
    void sendSecret(const MsgInfo& info)
}

template<typename Company>
class MsgSender{
public:
    void sendClear(const MsgInfo& info)
    {
        std::string msg;

        Company C;
        c.sendCleartext(msg);
    }
}

template<typename Company>
class LoggingMsgSender : public MsgSender<Company> {
public:
    void sendClearMsg(const MsgInfo& info)
    {
        sendClear(info);
    }
};
```
- **파생 클래스**에 있는 메시지 전송 함수의 이름(sendClearMsg)이 **기본 클래스**에 있는 것(sendClear)과 다른 꼼꼼하게 잘 된 설계이다.
  - [기본 클래스로부터 물려받은 이름을 파생 클래스에서 가리는 문제](/Chapter6/Item33.md)를 일으키지 않는다.
  - [상속받은 비가상 함수를 재정의하는 문제](/Chapter6/Item36.md)를 일으키지 않는다.
- `sendClear` 함수가 존재하지 않는다는 이유로 코드는 컴파일 되지 않는다.
  - 기본 클래스에 `sendClear` 함수가 있는데도 컴파일러는 기본 클래스를 보지 않는다.
- 컴파일러가 `LoggingMsgSender` 클래스 템플릿의 정의와 마주칠 때
  -  컴파일러는 이 클래스가 어디서 파생(`MsgSender<CompanyA>`? `MsgSender<CompanyB>`? `MsgSender<CompanyZ>`?)된 것인지 모른다. 
  - `MsgSender<Company>`인 것은 맞으나 `Company`는 **템플릿 매개변수**이고, 템플릿 매개변수는 인스턴스로 만들어질 때까지 무엇인지 알 수 없다.
- `CompanyZ`라면 `sendClear`함수는 존재하지 않기 때문에 이 코드는 말이 되지 않는다.
  - 함수 호출을 `C++`가 받아주지 않는다.

## 해결 방법
### 기본 클래스 함수에 대한 호출문 앞에 this->를 붙임
```cpp
template<typename Company>
class LoggingMsgSender : public MsgSender<Company> {
public:
    void sendClearMsg(const MsgInfo& info)
    {
       this->sendClear(info);
    }
};
```

### using 선언을 사용
```cpp
template<typename Company>
class LoggingMsgSender : public MsgSender<Company> {
public:
    using MsgSender<Company>::sendClear; // 컴파일러에게 sendClear 함수가 기본 클래스에 있다고 가정하라고 알려준다.
    void sendClearMsg(const MsgInfo& info)
    {
        sendClear(info);
    }
};
```

### 호출할 함수가 기본 클래스의 함수라는 점을 명시적으로 지정
```cpp
template<typename Company>
class LoggingMsgSender : public MsgSender<Company> {
public:
    void sendClearMsg(const MsgInfo& info)
    {
        MsgSender<Company>::sendClear(info);
    }
};
```
- 호출되는 함수가 **가상 함수**인 경우에는, **명시적 한정**을 해 버리면 **가상 함수 바인딩**이 무시되기 좋은 방법은 아니다.