# 항목 5. 컴파일러가 은근슬쩍 만드는 함수
- 컴파일러는 경우에 따라 클래스에 대해 아래 항목을 **암시적**으로 만들어 놓을 수 있다.  
    1. 기본 생성자 (선언되어 있지 않을때)
    2. 복사 생성자
    3. 복사 대입 연산자
    4. 소멸자
- 컴파일러가 만드는 함수의 형태는 모두 기본형이다. (`public` 멤버이며 `inline` 함수)

### 복사 대입 연산자의 자동 생성은 적법하고 이치에 닿아야만 한다.
- 둥 중 어느 검사도 통과하지 못하면 컴파일러는 `operator=`의 자동생성을 거부한다.
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
p = q; // 에러!
```