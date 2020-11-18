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