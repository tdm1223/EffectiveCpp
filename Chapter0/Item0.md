# 프리뷰
## 용어 사용
1. 선언(`declaration`) : 코드에 사용되는 어떤 대상의 **이름과 타입**을 컴파일러에게 알려주는 것
```cpp
std::size_t numDigits(int number); // 함수 선언

extern int x;                      // 객체 선언

class Widget;                      // 클래스 선언

template<typename T>
class GraphNode;                   // 템플릿 선언
```

2. 시그니처(`signature`) : 함수의 **매개변수 리스트**와 **반환 타입**
   - 위의 경우 `numDigits` 함수의 시그니처는 `std::size_t(int)`
3. 정의(`definition`) : 선언에서 빠진 **구체적인 세부사항**을 컴파일러에게 제공하는 것
   - 함수에 대한 정의는 함수에 대한 **코드 본문**을 제공하는것이다.
   - 클래스에 대한 정의는 클래스의 멤버를 넣어 준 결과이다.
4. 초기화(`initialization`) : 어떤 객체에 최초의 값을 부여하는 과정
   - 초기화는 **생성자**에 의해 이루어진다.

## explicit
- **암시적 형변환**이 일어나지 않도록 제한하는 키워드
```cpp
#include <iostream>
class A{
public:
   int num;
   A(int n) : num(n){};
};
void printA(A a){
   std::cout << a.num << std::endl;
}
int main(){
   int n = 26;
   printA(n);
}
```
- `printA(n)` 호출시 형변환이 발생하여 **A의 생성자**가 호출된다.
- 예상치 못한 형변환을 막으려면 생성자 앞에 `explicit` 키워드를 붙여준다.

## 복사 생성자, 복사 대입 연산자
```cpp
class Base{
public:
   Base(); // 기본 생성자
   Base(const Base& rhs); // 복사 생성자
   Base& operator=(const Base& rhs); // 복사 대입 연산자
}
```
- **복사 생성자**는 어떤 객체의 초기화를 위해 같은 타입의 객체로부터 초기화할 때 호출되는 함수이다.
- **복사 대입 연산자**는 같은 타입의 다른 객체에 어떤 객체의 값을 복사하는 용도로 쓰이는 함수이다.