## 항목 2. #define을 쓰려거든 const, enum, inline을 떠올리자
- 선행 처리자 보다 컴파일러를 가까이하자
- `#define ASPECT_RATIO 1.653`
    - **전처리 과정**을 통해 `ASPECT_RATIO`를 1.653으로 바꿔준다.
    - 값이 대체되는거라 에러가 발생하면 `ASPECT_RATIO`가 아닌 1.653만 보여 헷갈릴 수 있다.
- `const double AspectRatio = 1.653;`
    - 언어 차원에서 지원하는 **상수 타입**이기 때문에 컴파일러 눈에도 보리고 기호 테이블에도 들어간다.

### 클래스 상수
- 클래스 멤버로 상수를 정의하는 경우 상수를 **정적 멤버**로 만들어야 한다.
- 클래스 상수의 **선언**은 **헤더 파일**에 두고, **정의**는 **구현 파일**에 둔다.
- 정적 멤버로 사용하는 방식 해당 문법을 받아들이지 않는 컴파일러도 존재한다.
- `static const int NumTerns = 5`

### 나열자 둔갑술 (enum hack)
- enumerator 타입의 값은 int가 놓일 곳에도 쓸 수 있다.
- 나열자 둔갑술의 동작 방식은 `const` 보다는 `#define`에 가깝다.
    - `const`의 주소를 잡아낼 수는 있지만, `enum`의 주소를 취하는것은 불가능하다.
    - **클래스 멤버 변수로 상수**를 정의하는 경우 enum hack을 사용하면 좋다.

```cpp
class player{
private:
   enum { NumTurns = 5}; // enum hack. NumTurns를 5에 대한 기호식 이름으로
   int scores[NumTurns];
};
```