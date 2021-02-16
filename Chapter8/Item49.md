# 항목 49. new 처리자의 동작 원리를 제대로 이해하자
### operator new
- 객체 한 개를 할당할때 적용되는 함수이다.
- 배열을 담을 메모리의 경우 할당시 `operator new[]`를 써야 한다.

### new 처리자
- 사용자가 보낸 메모리 할당 요청을 `operator new` 함수가 맞추어 주지 못할 경우에 `operator new` 함수는 예외를 던진다.
- 예외를 던지기 전에, 사용자 쪽에서 지정할 수 있는 **에러 처리 함수**를 우선적으로 호출할 수 있다.
- 이 에러 처리 함수를 `new` 처리자 라고 한다.

### set_new_handler
- 메모리 고갈 상황을 처리할 함수를 **사용자 쪽에서 지정**할 수 있도록 표준 라이브러리에는 `set_new_handler`라는 함수가 있다.
```cpp
namespace std{
    typedef void (*new_handler)();
    new_handler set_new_handler(new_handler p) throw();
}
```
- `new_handler`는 받는 것도 없고 반환하는 것도 없는 함수 포인터에 대해 `typedef`를 걸어 놓은 **타입 동의어**이다.
- `set_new_handler`는 `new_handler`를 받고 `new_handler`를 반환하는 함수이다.
  - `new_handler`의 매개변수는 요구된 메모리를 `operator new`가 할당하지 못했을 때 `operator new`가 호출할 함수의 포인터이다.
  - `ner_Handler`의 반환값은 `set_new_handler`가 호출되기 바로 전까지 `new` 처리자로 쓰이고 있던 함수의 포인터이다.

```cpp
void outOfMem()
{
    std::cerr << "Unable to satisfy request for memory";
    std::abort();
}

int main()
{
    std::set_new_handler(outOfMem);
    int *bigDataArray = new int[100000000L];
}
```
- `operator new`가 1억 개의 정수 할당에 실패하면 `outOfMem` 함수가 호출된다.
- 이 함수는 **에러 메시지**를 출력하면서 프로그램을 강제로 끝낸다.
- 사용자가 부탁한 만큼의 메모리를 할당해 주지 못하면 `operator new`는 출분한 메모리를 찾아낼 때까지 `new` 처리자를 [되풀이해서 호출](/Chapter8/Item51.md)한다.

### new 처리자에서 해야할일
- 프로그램 동작에 좋은 영향을 미치는 쪽으로 `new` 처리자가 설계되어 있다면 아래 동작 중 하나를 꼭 해야 한다.
1. 사용할 수 있는 메모리를 더 많이 확보한다.
   - `operator new`가 시도하는 이후의 메모리 확보가 성공할 수 있도록 하는 전략
   - 프로그램이 시작할 때 메모리 블록을 크게 하나 할당해 놓았다가 `new` 처리자가 가장 처음 호출될 때 그 메모리를 쓸 수 있도록 허용하는 방법이 있다.
2. 다른 `new` 처리자를 설치한다.
   - 현재의 `new` 처리자가 더 이상 가용 메모리를 확보할 수 없다 해도, 자기 몫까지 해 줄 다른 `new` 처리자의 존재를 알고 있을 가능성이 있다.
   - 이런 상황에서 현재의 `new` 처리자는 제자리에서 다른 `new` 처리자를 설치한다.
3. `new` 처리자의 설치를 제거한다.
   - `set_new_handler`에 널 포인터를 넘긴다.
   - `new` 처리자가 설치된 것이 없으면, `operator new`는 메모리 할당이 실패했을 때 예외를 던진다.
4. 예외를 던진다.
   - `bac_alloc` 혹은 `bad_alloc`에서 파생된 타입의 **예외**를 던진다.
   - `operator new`에는 이쪽 종류의 에러를 받아서 처리하는 부분이 없기 때문에 이 예외는 메모리 할당을 요청한 원래의 위치로 전파된다.
5. 복귀하지 않는다.
   - `abort` 혹은 `exit`를 호출한다.

### 할당괸 객체의 클래스 타입에 따라서 메모리 할당 실패에 대한 처리를 다르게 하는 방법
- 클래스에서 자체 버전의 `set_new_handler` 및 `operator new`를 제공하도록 만들어 주면 된다.
- `set_new_handler`
  - 사용자로부터 그 클래스에 쓰기 위한 `new` 처리자를 받는다.
- `operator new`
  - 그 클래스 객체를 담을 메모리가 할당되려고 할 때 전역 `new` 처리자 대신 클래스 버전의 `new` 처리자가 호출되도록 만드는 역할