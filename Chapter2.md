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

