# 항목 28. 내부에서 사용하는 객체에 대한 '핸들'을 반환하는 코드는 되도록 피하자
### 자기모순적인 코드
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
    Point& upperLeft() const {return pData->ulhc;} // Point 객체에 대한 참조자 반환
    Point& lowerRight() const {return pData->lrhc;} // Point 객체에 대한 참조자 반환
private:
    std::shared_ptr<RectData> pData;
};

const Rectangle rec(coord1, coord2);
rec.upperLeft().setX(50); // 상수객체의 값을 바꿀수 있다.
```
- `upperLeft`를 호출한 쪽은 `rec`의 숨겨진 `Point` 데이터 멤버를 **참조자**로 끌어와 바꿀수 있는 문제가 발생한다.
    - rec은 **상수 객체**로 선언되어있다.

### 위 예제로 알 수 있는 점
- **클래스 데이터 멤버**는 그 멤버의 **참조자를 반환하는 함수**들의 **최대 접근도**에 따라 캡슐화 정도가 정해진다.
- 어떤 객체에서 호출한 상수 멤버 함수의 참조자 반환 값의 실제 데이터가 그 객체의 바깥에 저장되어 있다면, 함수 호출부에서 데이터의 수정이 가능하다.
- 참조자, 포인터 및 반복자는 모두 **핸들**이다.
    - 어떤 객체의 내부요소에 대한 **핸들을 반환**하게 만들면 언제든지 그 객체의 **캡슐화를 무너뜨리는 위험**이 존재한다.
- 객체의 내부요소에는 **데이터 멤버** 뿐만 아니라 일반적인 수단으로 접근이 불가능한 **멤버 함수**도 포함된다.

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