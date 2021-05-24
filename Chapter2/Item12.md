# 항목 12. 객체의 모든 부분을 빠짐없이 복사하자
## 복사 함수
1. 복사 생성자
2. 복사 대입 연산자

### 복사 함수 구현시 주의할 점
- 기존에 클래스를 설계할때 **복사 함수를 직접 구현**하는 경우가 있다.
- 작성 후 나중에 **변수가 추가**될때 **부분 복사**가 일어날 가능성이 존재한다.
```cpp
class B{
private:
    std::string name;
};

B::B(const B& rhs) : name(rhs.name){}

B& B::operator=(const B&rhs)
{
    name = rhs.name;
    return *this;
}
```
- 클래스에 멤버를 추가한다면, **추가한 데이터 멤버를 처리**하도록 **복사 함수**를 **재작성**해야 한다.
- 이런 상황에서 알려주는 컴파일러는 거의 없다.
- [경고 수준](/Chapter9/Item53.md)을 최대로 높여도 마찬가지이다.
- [생성자도](/Chapter1/Item4.md) [전부 갱신](/Chapter7/Item45.md)해줘야 하고 [비표준형 operator= 함수](/Chapter2/Item10.md)도 바꿔줘야 한다.
    
## 클래스 상속에서의 복사 함수
- **파생 클래스**에 대한 **복사 함수**를 직접 작성한다면 **기본 클래스** 부분을 복사에서 빠뜨리지 않도록 주의하여야 한다.
- 기본 클래스부분은 [private 멤버일 가능성이 높기 때문](/Chapter4/Item22.md)에 직접 건드리긴 어렵다.
- 파생 클래스의 복사 함수 안에서 **기본 클래스의 복사 함수를 호출**하도록 만든다.

```cpp
class Derived : public Base { };

Derived::Derived(const Derived& rhs) : Base(rhs) { } // 기본 클래스의 복사 생성자 호출

Derived& Derived::operator=(const Derived& rhs)
{
    Base::operator=(rhs); // 기본 클래스 부분을 대입
    return *this;
}
```