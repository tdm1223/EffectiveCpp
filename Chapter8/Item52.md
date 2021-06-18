# 항목 52. 위치지정 new를 작성한다면 위치지정 delete도 같이 준비하자
## new 표현식을 사용했을 때 호출되는 두 가지 함수
```cpp
Widget * pw = new Widget;
```
1. **메모리 할당**을 위해 `operator new` 호출
2. `Widget`의 **기본 생성자** 호출

### 생성자 호출이 진행되다가 예외가 발생한다면?
- 메모리 할당을 **취소**해야만 한다.
  - **메모리 누수**가 발생하기 때문이다.
- 사용자는 이 메모리를 해제할 수 없다.
  - `C++` 런타임 시스템에서 맡는다.

## C++ 런타임 시스템이 해 주어야 하는 일
- 호출한 `operator new` 함수와 짝이 되는 버전의 `operator delete` 함수를 호출한다.
   - `operator delete` 함수들 가운데 어떤 것을 호출해야 하는지 런타임 시스템이 제대로 알고 있어야 한다.
- 기본형 `operator new`는 기본형 `operator delete`와 짝을 맞춘다.
- 표준 형태의 `new` 및 `delete`만 사용하면 런타임 시스템은 `new`의 동작을 되돌릴 방법을 알고 있는 `delete`를 찾아내는 데 어려움이 없다.
  - 기본형이 아닌 형태를 선언한다면?

## 위치지정 new
- 기본형 `operator new` 함수와 달리 **매개변수를 추가로 받는 형태**로 선언한 `new`
- 위치지정 `new`는 가지각색일 수 있다.

### 어떤 객체를 생성시킬 메모리 위치를 나타내는 포인터를 매개변수로 받는 new
```cpp
void * operator new(std::size_t, void *pMemory) throw();
```
- `<new>`만 `#include` 하면 바로 사용할 수 있다.
- 위치지정 `new`의 원조이다.
- 이 버전의 `new` 함수는 표준 라이브러리의 여러 군데에서 쓰이고 있다.
  - `vector`의 미사용 공간에 원소 객체를 생성할 때 이 위치지정 `new`를 쓴다.

### 위치지정 new의 주의할 점
- `C++` 런타임 시스템은 호출된 `operator new`가 어떻게 동작하는지를 알아낼 방법이 없다.
  - 자신이 할당 자체를 되돌릴 수는 없다.
- 런타임 시스템은 호춮된 `operator new`가 받아들이는 **매개변수의 개수와 타입**이 똑같은 버전의 `operator delete`를 찾고 호출한다.
  - 짝을 이루는 `operator delete`가 존재해야 한다.(위치지정 `delete`라고 한다)

### 결론
- 위치지정 `new` 함수와 연관된 **메모리의 누수를**을 봉쇄하려면 표준 형태의 `operator delete`를 기본으로 마련해두어야 한다.
- 위치지정 `new`를 사용하였다면 똑같은 추가 매개변수를 받는 위치지정 `delete`도 존재해야 한다.

## 이름 가리기
- [바깥쪽 유효범위에 있는 어떤 함수의 이름과 클래스 멤버 함수의 이름이 같으면 바깥쪽 유효범위 함수가 이름만 같아도 가려진다.](/Chapter6/Item33.md)
  - 사용자 자신이 쓸 수 있다고 생각하는 다른 `new`(표준 버전 포함)를 클래스 전용의 `new`가 가리지 않도록 신경써야 한다.

### C++가 제공하는 전역 유효범위의 operator new의 형태 3가지 표준
```cpp
void *operator new(std::size_t) throw(std::bad_alloc);          // 기본형
void *operator new(std::size_t, void*) throw();                 // 위치지정
void *operator new(std::size_t, const std::nothrow_t&) throw(); // 예외불가
```
- 어떤 형태든 간에 `operator new`가 **클래스 안에 선언**되는 순간 표준 형태들이 전부 가려진다.
- 사용자가 표준 형태를 쓰지 못하게 막는 것이 의도한게 아니라면 표준 형태들도 사용자가 접근할 수 있도록 해야 한다.
- 간단한 방법으로는 기본 형태를 가지고 있는 기본 클래스 하나를 만드는 것이다.

### C++ 3가지 표준이 포함된 기본 클래스
```cpp
class StandardNewDeleteForms 
{
public: 
  // 기본형
  static void* operator new(std::size_t size) throw(std::bad_alloc)
  {
    return ::operator new(size);
  }
  static void operator delete(void* pMemory) throw()
  {
    ::operator delete(pMemory);
  }
  
  //위치지정
  static void* operator new(std::size_t size, void* ptr) throw()
  {
    return ::operator new(size, ptr);
  }
  static void operator delete(void* pMemory, void* ptr) throw()
  {
    ::operator delete(pMemory, ptr);
  }
  
  //예외불가
  static void* operator new(std::size_t size, const std::nothrow_t& nt) throw()
  {
    return ::operator new(size, nt);
  }
  static void operator delete(void* pMemory, const std::nothrow_t& nt) throw()
  {
    ::operator delete(pMemory);
  }
};
```
- 기본 클래스 하나를 만들고 `new` 및 `delete`의 기본 형태를 전부 넣어둔다.

### 표준 형태에 덧붙여 추가한 사용자 정의 형태
- 표준 형태에 덧붙여 사용자 정의 형태를 추가하고 싶다면 기본 클래스를 축으로 넓혀가면 된다.
- 상속과 `using` 선언을 사용하여 표준 형태를 파생 클래스쪽으로 끌어와 외부에서 사용할 수 있게 만든 후에 사용자 정의 형태를 선언한다.
```cpp
class Widget : public StandardNewDeleteForms 
{
public : 
  using StandardNewDeleteForms::operator new; 
  using StandardNewDeleteForms::operator delete;
  static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc);
  static void operator delete(void* pMemory, std::ostream& logStream) throw();
};
```