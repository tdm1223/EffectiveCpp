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