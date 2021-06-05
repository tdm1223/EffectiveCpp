# 항목 40. 다중 상속은 심사숙고해서 사용하자
## 다중 상속의 모호성
```cpp
class A{
public:
    void func();
};

class B{
private:
    bool func() const;
};

class D : public A, public B{};

D d;
d.func(); // B의 func()는 private 함수인데도 모호성 발생
```
- 두 `func` 함수들 중에서 파생 클래스가 접근할 수 있는 함수가 결정되는 것이 분명한데도 **모호성이 발생**한다.

![모호성](https://user-images.githubusercontent.com/21440957/120878128-68275680-c5f5-11eb-827a-9d367d571bd2.png)

## 죽음의 MI(multiple inheritance) 마름모꼴
```cpp
class File{};
class InputFile: public File{};
class OutputFile: public File{};
class IOFile: public InputFile, public OutputFile{};
```
- `File` 클래스 안에 `fileName`이라는 멤버 변수가 존재한다면 `IOFile` 클래스에는 기본적으로 **중복 생성된 fileName 멤버 변수**를 가진다.

## 가상 기본 클래스
- MI 마름모꼴을 해결방법은 **가상 기본 클래스**가 있다.
- 가상 기본 클래스로 삼을 클래스에 직접 연결된 파생 클래스에서 가상 상속을 사용하는것이다.
```cpp
class File{}; // 가상 기본 클래스
class InputFile: virtual public File{};
class OutputFile: virtual public File{};
class IOFile: public InputFile, public OutputFile{};
```
- 비용이 늘어나고 성능이 저하된다.
- 초기화 및 대입연산의 복잡도가 커지는 단점이 있다.
- 가상 기본 클래스는 사용하지 않는 것이 좋다.
- 가상 기본 클래스를 쓰지 않으면 안 될 상황이라면, 데이터를 넣지 않도록 신경 쓴다.

## 다중 상속을 적법하게 쓸 수 있는 경우
- **인터페이스 클래스**로부터 `public` 상속을 시키고 **구현을 돕는 클래스**로부터 `private` 상속을 시킨다.

```cpp
// 용도에 따라 구현될 인터페이스
class IPerson {
public:
    virtual ~IPerson();
    virtual string GetName() const = 0;
};

// IPerson 인터페이스를 구현하는 데 유용한 함수가 들어있는 클래스
class PersonInfo {
public:
    explicit PersonInfo(int pid);
    virtual ~PersonInfo();
    virtual const char* theName() const;
    virtual const char* valueDelimOpen() const;
    virtual const char* valueDelimClose() const;
};

class CPerson : public IPerson, private PersonInfo {
public:
    explicit CPerson(int pid) : PersonInfo(pid) {}

    // IPerson 클래스의 순수 가상 함수를 파생 클래스에서 구현
    virtual string GetName() const
    {
        return PersonInfo::theName();
    }
private:
    // 상속된 가상 함수들도 재정의 버전 구현
    const char* valueDelimOpen() const { return ""; }
    const char* valueDelimClose() const { return ""; }
};
```