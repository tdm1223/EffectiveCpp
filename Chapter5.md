# 구현

## 변수 정의는 늦출 수 있는 데까지 늦추기
- 생성자나 소멸자를 끌고 다니는 타입으로 변수를 정의하면 문게 되는 비용이 두개가 된다.
- 변수가 정의됐으나 사용되지 않은 경우에도 비용이 부과된다.
```cpp
std::string encryptPassword(const std::string& password)
{
    using namespace std;

    string encrypted; // string 변수 선언

    if (password.length() < MinimumPasswordLength)
    {
        throw logic_errod("Password is too short");
    }

    // 비밀번호 암호화 하는 로직

    return encrypted;
}
```
- 위 코드에서 `encrypted` 객체는 예외가 발생하면 사용되지 않는 객체이다.
- 따라서 해당 변수를 정의하는 일은 예외처리 아래까지 미루는 편이 낫다.

### 조금더 미루어보자
```cpp
std::string encryptPassword(const std::string& password)
{
    // 같은 로직

    string encrypted; // 기본 생성자에 의해 만들어지는 encrypted
    encrypted = password; // encrypted에 password를 대입

    encrypt(encrypted);
    return encrypted;
}
```
- 위 코드에서 바람직한 방법은 `encrypted`를 `password`로 초기화해 버리는 방법이다.
- 의미도 없고 비용도 만만치 않을 듯한 **기본 생성자** 호출을 건너뛸수 있게 된다.

### 조금더 미루어보자2
```cpp
std::string encryptPassword(const std::string& password)
{
    // 같은 로직

    string encrypted(password); // 정의와 동시에 초기화

    encrypt(encrypted);
    return encrypted;
}
```
- 위처럼정의와 동시에 초기화 함으로써 성능을 높일 수 있고 프로그램도 깔끔해 진다.

## 캐스팅은 절약, 또 절약
- 정말 잘 작성된 C++ 코드는 캐스팅을 거의 쓰지 않는다.

### 네 가지 캐스트 연산자
1. `const_cast`
    - 상수성을 없애는 용도
2. `dynamic_cast`
    - 안전한 다운 캐스팅
3. `reinterpret_cast`
    - 하부 수준 캐스팅
4. `static_cast`
    - 암시적 변환

### 캐스팅을 써야할 경우
- 객체를 인자로 받는 함수에 객체를 넘기기 위해 **명시 호출 생성자**를 호출하고 싶을 경우
- 캐스팅이 어쩔 수 없이 필요하다면, 함수 안에 숨길수 있도록 하는것이 좋다.
- 캐스트 연산자가 입맛 당기는 상황이라면 뭔가 꼬여가는 징조다.

### 캐스팅은 다른 타입으로 처리하라고 컴파일러에게 알려주는것이 아니다.
```cpp
class Base{};
class Derived:public Base{};
Derived d;
Base* pb = &d;
```
- 포인터의 `offset`을 `Derived*` 포인터에 적용하여 실제의 `Base*` 포인터 값을 구하는 동작이 런타임에 이루어진다.
- 객체 하나가 가질 수 있는 주소가 한개가 아니라 그 이상이 될 수 있음을 보여주는 사례가 된다.

### dynamic_cast
- 상당수의 구현환경에서 이 연산자가 정말 느리게 구현되어 있다.
- **파생 클래스** 객체임이 분명해 **파생 클래스**의 함수를 호출하고 싶은데, 그 객체를 조작할 수 있는 수단으로 **기본 클래스**의 포인터밖에 없을 경우 사용하게 된다.
- 해결 방법
    1. 파생 클래스 객체에 대한 포인터를 컨테이너에 담아줌으로써 각 객체를 기본 클래스 인터페이스를 통해 조작할 필요를 제거한다.
    2. 원하는 조작을 가상 함수 집합으로 정리해서 기본 클래스에 넣어둔다.

## 내부에서 사용하는 객체에 대한 '핸들'을 반환하는 코드는 피하기
```cpp
// 점을 나타내는 클래스
class Point{
public:
    Point(int x, int y);
    void SetX(int newVal);
    void SetY(int newVal);
};

struct RectData{
    Point ulhc;
    Point lrhc;
};

class Rectangle{
public:
    Point& upperLeft() const {return pData->ulhc;}
    Point& lowerRight() const {return pData->lrhc;}
private:
    std::shared_ptr<RectData> pData;
};

const Rectangle rec(coord1, coord2);
rec.upperLeft().setX(50);
```
- upperLeft를 호출한 쪽은 rec의 숨겨진 Point 데이터 멤버를 참조자로 끌어와 바꿀수 있는 문제가 발생한다.
    - rec은 상수 객체로 선언되어있다.

### 위 예제로 알 수 있는 점
1. 클래스 데이터 멤버는 그 멤버의 참조자를 반환하는 함수들의 최대 접근도에 따라 캡슐화 정도가 정해진다.
2. 어떤 객체에서 호출한 상수 멤버 함수의 참조자 반환 값의 실제 데이터가 그 객체의 바깥에 저장되어 있다면, 이 함수의 호출부에서 그 데이터의 수정이 가능하다.


- 참조자, 포인터 및 반복자는 모두 핸들이다.
    - 어떤 객체의 내부요소에 대한 핸들을 반환하게 만들면 언제든지 그 객체의 캡슐화를 무너뜨리는 위험이 존재한다.
- 어떤 객체의 내부요소에는 데이터 멤버 뿐만 아니라 일반적인 수단으로 접근이 불가능한 멤버 함수도 내부요소에 포함된다.

### 해결책
```cpp
const Point& upperLeft() const {return pData->ulhc;}
const Point& lowerRight() const {return pData->lrhc;}
```
- upperLeft(), lowerRight() 반환 타입에 const 키워드를 붙여준다.
    - 사용자는 사각형을 정의하는 꼭짓점 쌍을 읽을 수는 있지만 쓸 수는 없게 된다.

### upperLeft 함수와 lowerRight 함수에서 내부 데이터에 대한 핸들을 반환하고 있는 부분이 남아있다.
```cpp
class GUIObject{};
const Rectanble boundingBox(const GUIObject& obj);

GUIObject * pgo;
const Point *pUpperLeft = &(boundingBox(*pgo).upperLeft());
```
- 마지막 문장의 흐름
    1. `boundingBox`함수 호출시 `Rectangle` 임시 객체 생성
    2. 생성된 임시 객체에 대해 `upperLeft` 호출 (`Point` 객체중 하나에 대한 참조자가 나옴)
    3. 위에서 나온 참조자에 & 연산자를 건 결과 값(주소)이 `pUpperLeft` 포인터에 대입
    4. 문장이 끝날 무렵, `boundingBox`함수의 반환 값(임시 객체)이 소멸
    5. 임시 객체가 소멸되니 그 안에 들어 있는 Point 객체들도 덩달아 소멸
    6. `pUpperLeft` 포인터가 가리키는 객체는 없어진다.
- `pUpperLeft`에게 객체를 달아 줬다가(생성된 임시 객체) **주소 값**만 남기고 모두 빼앗아 간 꼴이 된다.
- 위의 코드처럼 객체의 내부에 대한 핸들을 반환하는 함수는 위험하다.