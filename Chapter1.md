## C++를 언어들의 연합체로 바라보는 안목
1. C
2. 객체 지향 개념의 C++
3. 템플릿 C++
4. STL

## #deine을 쓰려거든 const, enum, inline을 떠올리자
- 선행 처리자 보다 컴파일러를 가까이하자
- `#define ASPECT_RATIO 1.653 `
    - 전처리 과정을 통해 ASPECT_RATIO를 1.653으로 바꿔준다.
    - 값이 대체되는거라 에러가 발생하면 ASPECT_RATIO가 아닌 1.653만 보여 헷갈릴 수 있다.
- `const double AspectRatio = 1.653;`
    - 언어 차원에서 지원하는 상수 타입이기 때문에 컴파일러 눈에도 보리고 기호 테이블에도 들어간다.

- 클래스 멤버로 상수를 정의하는 경우 상수를 정적 멤버로 만들어야 한다.

### enum hack(나열자 둔갑술)
- enumerator 타입의 값은 int가 놓일 곳에도 쓸 수 있다.
- enum hack의 동작 방식은 const 보다는 #define에 가깝다.
    - const의 주소를 잡아낼 수는 있지만, enum의 주소를 취하는것은 불가능하다.
    - 클래스 멤버 변수로 상수를 정의하는 경우 enum hack을 사용하면 좋다.

```cpp
class player{
private:
   enum { NumTurns = 5}; // enum hack. NumTurns를 5에 대한 기호식 이름으로
   int scores[NumTurns];
}
```

- `static const int NumTurns = 5;`
    - 정적 멤버로 사용하는 방식 해당 문법을 받아들이지 않는 컴파일러도 존재한다.

## 낌새만 보이면 const를 들이대 보자