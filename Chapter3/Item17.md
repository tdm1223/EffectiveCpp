# 항목 17. new로 생성한 객체를 스마트 포인터에 저장하는 코드는 별도의 한 문장으로 만들자
### 자원을 흘릴 가능성이 있는 코드
```cpp
processWidget(std::shared_ptr<Widget>(new Widget), priority());
```
- `processWidget` 호출 코드를 만들기 전에 **컴파일러**는 아래 **세가지 연산**을 위한 코드를 만들어야 한다.
    1. `new Widget` 표현식을 실행하는 부분
    2. `shared_ptr` 생성자를 호출하는 부분
    3. `priority()`를 호출하는 부분
- **자원 관리 객체**를 쓰고 있지만 자원을 **흘릴 가능성**이 있다.

### 원인
- 연산의 실행 순서가 컴파일러마다 다르다.
- `new Widget` -> `priority()` -> `shared_ptr`순으로 호출이 되고 `priority()`에서 예외가 발생한다면 `new widget`으로 만들어졌던 **포인터가 유실**될 수 있다.
- **자원이 생성되는 시점**과 **자원이 자원 관리 객체로 넘어가는 시점** 사이에 예외가 끼어들 수 있다.

### 해결 방법
- 스마트 포인터에 저장하는 코드를 별도의 문장 하나로 만들고, 스마트 포인터를 `processWidget`에 넘긴다.
```cpp
std::shared_ptr<Widget> pw(new Widget);

processWidget(pw, priority());
```
- 컴파일러의 재조정을 받을 여지가 적어 자원 누출 가능성이 없다.