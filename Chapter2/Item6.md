# 항목 6. 컴파일러가 만들어낸 함수가 필요없다면 확실히 이들의 사용을 금해버리자
- 세상에 하나밖에 없는 객체가 있다면 그 객체는 사본을 만들 수 없어야 한다.
- 컴파일러가 **복사 생성자** 및 **복사 대입 연산자**를 [은근슬쩍 만들기 때문에](/Chapter2/Item5.md) **복사의 가능성이 존재**한다. 

### 복사를 막는 방법
1. **복사 생성자** 및 **복사 대입 연산자**를 `private` 멤버로 선언한다.
    - 여전히 그 클래스의 **멤버 함수** 및 **friend 함수**가 호출할 수 있는 위험이 존재한다.
2. 해당 함수들을 클래스에 **선언**만 한다.
    - 선언만 하고 정의를 하지 않는다.
    - 객체의 복사를 시도하려고 할때 컴파일러가 못하도록 막는다.

```cpp
class test
{
public:
    test();
private:
    // 선언만 존재하는 복사 생성자와 복사 대입 연산자
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

3. 선언만한 복사 생성자와 복사 대입연산자를 해당 클래스에 넣지 말고 별도의 **기본 클래스**에 넣고 해당 **클래스로부터 파생**시킨다.
```cpp
class Uncopyable{
protected:
    Uncopyable() {} // 생성 허용
    ~Uncopyable() {} // 소멸 허용
private:
    Uncopyable(const Uncopyable&);
    Uncopyable& operator=(const Uncopyable&);
};

class test : private Uncopyable{};
```
-  `public` 상속을 [받을 필요가](/Chapter6/Item32.md) [없다.](/Chapter6/Item39.md)
- Uncopyable의 소멸자는 [가상 소멸자가 아니어도 된다.](/Chapter2/Item7.md)
- [다중 상속의 문제](/Chapter6/Item40.md)가 발생할 수 있는데, 다중 상속 시 [공백 기본 클래스 최적화](/Chapter6/Item39.md)가 돌아가지 못할때가 있지만 무시해도 상관없다.