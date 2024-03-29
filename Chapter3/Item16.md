# 항목 16. new 및 delete를 사용할 때는 형태를 반드시 맞추자
## new와 delete를 잘못 사용한 코드
```cpp
std::string* stringArray = new std::string[100];
delete stringArray;
```
- 100개의 `string` 객체들 가운데 99개는 정상적인 소멸 과정을 거치지 못할 가능성이 크다.
- `new` 표현식에 `[]`를 썼으면, 대응되는 `delete` 표현식에도 `[]`를 써야 한다.
- `std`에는 `string`, `vector`같은 클래스 템플릿이 있어서 잘 활용하면 동적 할당 배열이 필요해질 경우가 거의없다. (`vector<string>`사용)

## new 연산자를 사용해서 표현식을 쓸 경우
1. [메모리](/Chapter8/Item49.md) [할당](/chapter8/Item51.md)
2. 할당된 메모리에 대해 **한 개 이상의 생성자**가 호출된다.

## delete 연산자를 사용해서 표현식을 쓸 경우
1. 기존에 할당된 메모리에 대해 **한 개 이상의 소멸자** 호출된다.
2. [메모리](/Chapter8/Item49.md) [해제](/chapter8/Item51.md)
