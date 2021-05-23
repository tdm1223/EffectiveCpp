# 항목 10. 대입 연산자는 *this의 참조자를 반환하게 하자
## C++의 대입연산
```cpp
int x, y, z;
x = y = z = 15;
```
- `C++`의 대입연산은 사슬처럼 엮일 수 있다.
- 대입 연산은 우측 연관 연산이고 위 연산 사슬은 아래와 같이 분석된다.
```cpp
x = (y = (z = 15));
```

## C++ 대입 연산자 관례
- 대입 연산자에 `*this`를 통해 좌변 객체(의 참조자)를 반환하도록 하는것이 **관례**이다.
```cpp
Widget& operator=(const Widget& rhs)
{
    return *this; // 좌변 객체(의 참조자)를 반환!
}

Widget& operator+=(const Widget& rhs)
{
    return *this; // 단순 대입 연산이 아니어도 동일한 규약이 적용된다!
}
```
- [표준 라이브러리](/Chapter9/Item54.md)에 속한 모든 타입에서도 따르고 있다.