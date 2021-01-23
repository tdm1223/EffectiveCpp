# 항목 10. 대입 연산자는 *this의 참조자를 반환하게 하자
### C++의 대입연산
- `x = y = z = 15`와 같이 사슬처럼 엮일 수 있다.
    - `x = (y = (z = 15));`
- 대입 연산자는 **우측 연관 연산**이다.

### 대입연산자 관례
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