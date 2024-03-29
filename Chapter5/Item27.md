# 항목 27. 캐스팅은 절약, 또 절약! 잊지 말자
## 캐스팅 문법
### C 스타일의 캐스트
- (T) 표현식

### 함수 방식 캐스트
- T(표현식)

### C++ 스타일의 캐스트
- 구형 스타일의 캐스트보단 `C++` 스타일의 캐스트를 사용하는것이 좋다.
    - 발견하기 쉽고 설계자가 어떤 역할을 의도했는지 자세하게 드러난다.

## 네 가지 캐스트 연산자
### const_cast
- 상수성을 없애는 용도

### dynamic_cast
- 안전한 다운 캐스팅
- 런타임 비용이 높다.

### reinterpret_cast
- 하부 수준 캐스팅

### static_cast
- 암시적 변환 ([비상수 객체를 상수 객체로](/Chapter1/Item3.md))

## 캐스팅을 써야할 경우
- 객체를 인자로 받는 함수에 객체를 넘기기 위해 **명시 호출 생성자**를 호출하고 싶을 경우
- 캐스팅이 어쩔 수 없이 필요하다면, 함수 안에 숨길수 있도록 하는것이 좋다.
- 캐스팅이 들어가면 보기엔 맞는 것 같지만 실제로는 틀린 코드를 쓰고도 모르는 경우가 많아진다.
    - 캐스트 연산자가 입맛 당기는 상황이라면 뭔가 꼬여가는 징조다.

## 캐스팅은 다른 타입으로 처리하라고 컴파일러에게 알려주는것이 아니다.
```cpp
class Base{};
class Derived:public Base{};
Derived d;
Base* pb = &d;
```
- 포인터의 `offset`을 `Derived*` 포인터에 적용하여 실제의 `Base*` 포인터 값을 구하는 동작이 **런타임**에 이루어진다.
- 객체 하나가 가질 수 있는 주소가 한개가 아니라 그 이상이 될 수 있음을 보여주는 사례가 된다.

## dynamic_cast
- 상당수의 구현환경에서 이 연산자가 느리게 구현되어 있다.
- **파생 클래스** 객체임이 분명해 **파생 클래스**의 함수를 호출하고 싶은데, 그 객체를 조작할 수 있는 수단으로 **기본 클래스**의 포인터밖에 없을 경우 사용하게 된다.

### 회피 방법
1. **파생 클래스 객체**에 대한 포인터를 컨테이너에 담아줌으로써 각 객체를 기본 클래스 인터페이스를 통해 조작할 필요를 제거한다.
```cpp
typedef std::vector<std::shared_ptr<SpecialWindow>> VPSW;
VPSW winPtrs;
for(VPSW::iterator iter = sinPtrs.egin(); iter!= winPtrs.end(); ++iter)
{
    (*iter)->blink();
}
```
2. 원하는 조작을 가상 함수 집합으로 정리해서 기본 클래스에 넣어둔다.
```cpp
class Window{
public:
    virtual void blink() {}
};

class SpecialWindow: public Window{
public:
    virtual void blink() { ... }
}

typedef std::vector<std::shared_ptr<Window>>> VPW;
VPW winPtrs;
for(VPW::iterator iter = sinPtrs.egin(); iter!= winPtrs.end(); ++iter)
{
    (*iter)->blink();
}
```