# 프리뷰
## 용어 사용
1. 선언(declaration) : 코드에 사용되는 어떤 대상의 이름과 타입을 컴파일러에게 알려주는 것
   - `std::size_t numDigits(int number);` : 함수 선언
2. 시그니처(signature) : 함수의 매개변수 리스트와 반환 타입
   - 위의 경우 `numDigits` 함수의 시그니처는 `std::size_t(int)`
3. 정의(definition) : 선언에서 빠진 구체적인 세부사항을 컴파일러에게 제공하는 것
   - 함수에 대한 정의는 함수에 대한 코드 본문을 제공하는것이다.
   - 클래스에 대한 정의는 클래스의 멤버를 넣어 준 결과이다.
4. 초기화(initialization) : 어떤 객체에 최초의 값을 부여하는 과정
   - 초기화는 생정자에 의해 이루어진다.

## keyword explicit
- 암시적 형변환이 일어나지 않도록 제한하는 키워드
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
- `printA(n)` 호출시 형변환이 발생하여 A의 생성자가 호출된다.
- 예상치 못한 형변환을 막으려면 생성자 앞에 `explicit` 키워드를 붙여준다.
- 확인 결과 알약 코드에도 존재하기는 하더라..

## 복사 생성자, 복사 대입 연산자
```cpp
class Widget{
public:
   Widget(); // 기본 생성자
   Widget(const Widget& rhs); // 복사 생성자
   Widget& operator=(const Widget& rhs); // 복사 대입 연산자
}
```
- 복사 생성자는 값에 의한 객체 전달을 정의해 준다.