# 항목 14. 자원 관리 클래스의 복사 동작에 대해 진지하게 고찰하자
- 힙에 생기지 않는 자원은 `auto_ptr` 혹은 `shared_ptr`등의 스마트 포인터로 처리해 주기엔 맞지 않다.
- 자원 관리 클래스를 직접 만들어야 하는 경우도 존재한다.
- 아래는 뮤텍스 잠금을 관리하는 직접 생성한 자원 관리 클래스이다.
```cpp
class Lock{
public:
    explicit Lock(Mutex *pm)
    : mutexPtr(pm)
    {
        lock(mutexPtr);     // 자원 획득
    }
    ~Lock()
    {
        unlock (mutexPtr);  // 자원 해제
    }
private:
    Mutex *mutexPtr;
};
```
- `Lock`객체가 복사된다면 어떻게 되어야 할까?

## RAII 객체가 복사될 때의 동작
### 복사를 금지한다.
```cpp
class Lock : private Uncopyable{ // 복사 금지
public:
}
```
- `RAII` 객체가 복사되는 것 자체가 말이 안되는 경우가 많다.
- 복사를 막는것은 [복사 연산을 private 멤버로 만드는 방법](/Chapter2/Item6.md)을 사용하면 된다.

### 관리하고 있는 자원에 대해 참조 카운팅을 수행한다.
```cpp
class Lock
{
public:
    explicit Lock(Mutex *pm) : mutexPtr(pm, unlock) // 삭제자로 unlock 함수 사용
    {
        lock(mutexPtr.get());
    }
private:
    std::shared_ptr<Mutex> mutexPtr; // 포인터 대신 shared_ptr을 사용
}
```
- 자원을 참조하는 객체의 개수에 대한 **카운트를 증가**시키는 식으로 `RAII` 객체의 복사 동작을 만든다.
- `shared_ptr`에서 **삭제자를 지정**한다.
- `Lock` 클래스의 **소멸자**를 선언하지 않는다.(필요없기 때문)
- `mutexPtr`의 **소멸자**는 뮤텍스의 참조 카운트가 0이 될 때 `shared_ptr`의 삭제자를 자동으로 호출한다.

### 관리하고 있는 자원을 진짜로 복사한다.
- 자원 관리 객체를 복사하면 객체가 둘러싸고 있는 자원까지 복사되어야 한다.(**깊은 복사**)

### 관리하고 있는 자원의 소유권을 옮긴다.
- `auto_ptr`의 복사 동작처럼 소유권을 사본으로 옮긴다.