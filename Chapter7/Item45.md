# 항목 45. "호환되는 모든 타입"을 받아들이는 데는 멤버 함수 템플릿이 직방!
## 스마트 포인터로 대신할 수 없는 포인터의 특징
- 암시적 변환 지원
- **파생 클래스 포인터**는 **기본 클래스 포인터**로의 암시적 변환이 가능하다.
- **비상수 객체에 대한 포인터**는 **상수 객체에 대한 포인터**로의 암시적 변환이 가능하다.
```cpp
class Top{};
class Middle : public Top{};
class Bottom : public Middle {};
Top * pt1 = new Middle; // Middle* => Top*의 변환
Top * pt2 = new Bottom; // Bottom* => Top*의 변환
const Top *pct = pt1;   // Top* => const Top*의 변환
```

## 멤버 함수 템플릿
- 클래스의 **멤버 함수**를 찍어내는 템플릿
```cpp
template<typename T>
class SmartPtr{
public:
    // 일반화된 복사 생성자를 만들기 위해 마련한 멤버 템플릿
    template<typename U>
    SmartPtr(const SmartPtr<U>& other);
};
```
- 모든 **T 타입** 및 모든 **U 타입**에 대해서 `SmartPtr<T>` 객체가 `SmartPtr<U>`로 부터 생성될 수 있다.
    - `SmartPtr<U>`의 참조자를 매개변수로 받아들이는 생성자가 `SmartPtr<T>` 안에 들어있다.
    - **일반화 복사 생성자**라고 부른다.
- 일반화 복사 생성자는 `explicit`로 선언되지 않았다.
  - 스마트 포인터도 기본제공 포인터처럼 **암시적 변환**이 가능하는 형태로 동작하도록 흉내내기 위해서다.
- [public 상속의 의미](/Chapter6/Item32.md)를 역행하게 된다.
  - `SmartPtr<Top>`으로부터` SmartPtr<Bottom>`을 만들 수 있다.
- `SmartPtr<double>`로 부터 `SmartPtr<int>`를 만들 수도 있다.
  - 이에 대응되는 `int*`에서 `double*`로 진행되는 암시적 변환은 불가능 하다.
- 타입 변환에 대한 제약이 필요하다.

## 타입 변환 제약 두기
- [get 멤버 함수를 통해 해당 스마트 포인터 객체에 자체적으로 담긴 기본제공 포인터의 사본을 반환한다고 가정한다.](/Chapter3/Item15.md)
    - 생성자 템플릿에 원하는 **타입 변환 제약**을 줄 수 있다.
```cpp
template<typename T>
class SmartPtr{
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other) // 이 SmartPtr에 담긴 포인터를
    : heldPtr(other.get()) {}          // 다른 SmartPtr에 담긴 포인터로 초기화한다.
    T * get() const {return heldPtr;}
private:
    T *heldPtr; // SmartPtr에 담긴 기본 제공 포인터
}
```
- **암시적 변환이 가능**할 때만 컴파일 에러가 나지 않는다.
    - `SmartPtr<T>`의 일반화 복사 생성자는 호환되는 타입의 매개변수를 넘겨받을 때만 컴파일 되게 된다.
- 어떤 클래스의 복사 생성을 전부 컨트롤 하고자 한다면 **일반화 복사 생성자**는 물론이고 **보통의 복사 생성자**까지 직접 선언해야 한다.
```cpp
template<class T> class shared_ptr{
public:
    shared_ptr(shared_ptr const& r);               // 복사 생성자

    template<class Y>                              // 일반화
    shared_ptr(shared_ptr<T> const& r);            // 복사 생성자

    shared_ptr& operator=(shared_ptr const& r);    // 복사 대입 연산자

    template<class Y>                              // 일반화
    shared_ptr& operator=(shared_ptr<Y> const& r); // 복사 대입 연산자
}
```