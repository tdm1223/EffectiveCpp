# 항목 13. 자원관리에는 객체가 그만
### 누수가 일어날 수 있는 코드
```cpp
void f()
{
    Investment *pInv = createInvestment(); // 파생된 클래스의 객체를 얻어내는 팩토리 함수
    // return? goto? continue? exception?
    delete pInv;
}
```
- `createInvestment()` 함수로 부터 얻은 **객체의 삭제를 실패**할 수 있는 경우가 존재한다.
- `return`, `goto`, `continue`, `예외발생`등의 경우를 통해 삭제가 이루어지지않고 함수를 빠져나올수가 있다.
- 제대로 종료되지 않는다면 **메모리 누수**가 일어나고, 그 객체가 갖고 있던 자원까지 모두 샌다.
- 얻어낸 자원이 항상 해제되도록 하려면, 자원을 객체에 넣고 그 **자원 해제**를 **소멸자**가 맡도록 하며, **소멸자**는 **실행제어**가 `f`를 떠날 때 호출되도록 만드는 것이다.

### auto ptr
```cpp
void f()
{
    std::auto_ptr<Investment> pInv(createInvestment()); // 팩토리 함수 호출
}
```

#### 특징
1. 자원을 획득한 후에 **자원 관리 객체**에게 넘긴다.
    - RAII(Resource Acquisition Is Initialization)라는 이름으로 통용된다.
2. **자원 관리 객체**는 자신의 소멸자를 사용해서 자원이 **확실히 해제**되도록 한다.

#### 주의해야 할 점
- `auto_ptr`은 자신이 소멸될 때 가리키고 있는 대상에 대해 자동으로 `delete`를 실행한다.
    - 객체를 가리키는 `auto_ptr`의 개수가 둘 이상이면 절대로 안된다. (두번 삭제가 발생)
- `auto_ptr`로 객체를 복사하면 원본 객체를 `null`로 만든다.
- STL 컨테이너의 경우엔 원소들이 정상적인 복사 동작을 가져야 하기 떄문에, `auto_ptr`은 원소로 허용되지 않는다.

### 참조 카운팅 방식 스마트 포인터(Reference-Counting Smart Pointer: RCSP)
- `auto_ptr`을 사용할 수 없는 상황일때 사용하기 좋은 스마트 포인터이다.
- `RCSP`는 자원을 가리키는 **외부 객체의 개수를 유지**하고 있다가 개수가 **0이 되면 해당 자원을 자동으로 삭제**한다.
- 동작이 **가비지 컬렉션**과 유사하다.
- 참조 상태가 고리를 이루는 경우는 없앨 수 없다.(가비지 컬렉션과 다른점)
- [shared_ptr](/Chapter9/Item54.md)이 대표적인 `RCSP`이다.

```cpp
void f()
{
    std::shared_ptr<Investment> pInv(createInvestment()); // 팩토리 함수 호출
}
```

### 스마트포인터 주의사항
- `delete[]` 연산자가 아닌 `delete`연산자를 사용하기 때문에 동적으로 할당한 배열에 대해 `auto_ptr`이나 `shared_ptr`을 사용하면 안된다.
- **동적 배열**을 썼을 때 컴파일 에러가 나지 않기 때문에 더 주의해야 한다.