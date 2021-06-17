# 항목 51. new 및 delete를 작성할 때 따라야 할 기존의 관례를 잘 알아 두자
## operator new 작성 규칙
- 반환 값이 제대로 되어 있어야 한다.
  - 요청된 메모리를 마련해 줄 수 있을 경우 반환값은 메모리에 대한 포인터이다.
  - 마련할 수 없는 경우 `bad_alloc`타입의 예외를 던진다.
- 가용 메모리가 부족할 경우에는 [new 처리자 함수](/Chapter8/Item49.md)를 호출해야 한다.
- 크기가 없는 메모리 요청(0바이트)에 대한 대비책을 갖춰두어야 한다.
- [기본 형태의 new가 가려지지 않도록](/Chapter8/Item52.md) 한다.

## 비멤버 버전의 operator new 함수 의사코드
- `operator new` 멤버 함수는 파생 클래스 쪽으로 상속이 되는 함수이다.
```cpp
void * operator new(std::size_t) throw(std::bac_alloc) // 다른 매개변수를 추가로 가질 수 있다.
{
    using namespace std;
    if (size == 0)
    {
        size = 1; // 0 바이트 요청이 들어오면 1 바이트 요구로 간주하고 처리
    }

    while(true)
    {
        size 바이트 할당
        if (할당 성공)
            return (할당된 메모리에 대한 포인터)

        // 할당이 실패했을 경우, 현재의 new 처리자 함수가 어느 것으로 설정되어 있는지 찾는다.
        new_handler globalHandler = set_new_handler(0);
        set_new_handler(globalHandler);

        if(globalHandler) (*globalHandler)();
        else throw std::bac_alloc();
    }
}
```
### while(true)문을 빠져나오는 방법
- **메모리 할당**이 성공한다.
- [new 처리자에서 해야할일](/Chapter7/Item47.md)중 한가지를 `new` 처리자 함수쪽에서 해준다.
  - `new` 처리자에서 직무유기를 해버릴 경우 `operator new`의 내부 루프는 걸대로 스스로 끝나지 않는다.

## operator new 클래스 전용 버전
```cpp
class Base{
public:
    static void * operator new(std::size_t size) throw(std::bad_alloc);
};

class Derived : public Base {};
derived * p = new Derived;

void * Base::operator new (std::size_t size) throw(std::bad_alloc)
{
    if (size != sizeof(base))
    {
        return ::operator new(size); // 틀린 크기가 들어오면 표준 operator new 쪽에서 메모리 할당 요구를 처리한다.
    }

    // 맞는 크기가 들어오면 메모리 할당 요구를 여기서 처리한다.
}
```
- `sizeof(Base)`와 `size`를 비교하는 코드에서 **0바이트 점검**도 함께 진행한다.
- `size`가 0이면 `if` 문이 거짓이 되어 메모리 처리 요구가 `::operator new` 쪽으로 넘어간다.
- 배열에 대한 메모리 할당을 클래스 전용 방식으로 하고 싶다면 `operator new[]` 함수를 구현하면 된다.

## operator delete 작성 규칙
- `C++`는 널 포인터에 대한 `delete` 적용이 **항상 안전하도록 보장**한다.
```cpp
void operator delete(void *rawMemory) throw()
{
    if(rawMemory == 0) return; // 널 포인터가 delete되려고 할 경우에는 아무것도 하지 않는다.

    // rawMemory가 가리키는 메모리 해제
}
```

## operator delete 클래스 전용 버전
- 삭제될 메모리의 **크기를 점검**하는 코드를 넣어 준다.

```cpp
class Base{
public:
    static void operator delete(void * rawMemory, std::size_t size) throw();
};

void Base::operator delete(void *rawMemory, std::size_t size) throw()
{
    if (rawMemory == 0) return; // 널 포인터 점검

    if (size != sizeof(Base)) // 크기가 틀린 경우 표준 operator delete가 메모리 삭제 요청을 맡는다.
    {
        ::operator delete(rawMemory);
        return;
    }

    // rawMemory가 가리키는 메모리를 해제한다.

    return;
}
```
- 가상 소멸자가 없는 기본 클래스로부터 파생된 클래스의 객체를 삭제하려고 할 경우에는 `operator delete`로 `C++`가 넘기는 `size_t` 값이 엉터리일 수 있다.
- 기본 클래스에서 가상 소멸자를 빼먹으면 `operator delete` 함수가 [올바르게 동작하지 않을 수 있다.](/Chapter2/Item7.md)
