# 항목 11. operator=에서는 자기대입에 대한 처리가 빠지지 않도록 하자
## 자기대입
- 어떤 객체가 **자기 자신**에 대해 대입 연산자를 적용하는것
```cpp
class Widget{};
Widget w;
w = w; // 자기에 대한 대입
```
- 여러 곳에서 하나의 객체를 참조하는 상태인 **중복 참조**(`aliasing`) 때문에 자기대입이 생길 수 있다.

## 해결책
### 일치성 검사
```cpp
Wodget& Widget::operator=(const Widget& rhs)
{
    if (this == &rhs) return *this; // 자기대입인지 검사

    // 대입 연산 처리
}
```
- `operator=`의 첫머리에서 **자기대입을 점검**한다.
- 대입 연산자에서 **메모리 할당**이 일어날 경우 자기대입을 점검하는 방법으론 한계가 존재한다.

### 문장의 실행 순서 조정
```cpp
Widget& Widget::operator=(const Widget& rhs)
{
    Bitmap *pOrig  = pb;      // 원래의 객체를 기억
    pb = new Bitmap(*rhs.pb); // 사본을 가리키게 만든다.
    delete pOrig;             // 원래의 객체 삭제
    return *this;
}
```
1. 원래의 객체를 어딘가에 기억해둔다.
2. 사본을 가리키게 만든다.
3. 원래의 객체를 삭제한다.

### [복사 후 맞바꾸기](/Chapter5/Item29.md)
```cpp
class Widget{
public:
    void swap(Widget& rhs);
};

Widget& Widget::operator=(const Widget& rhs)
{
    Widget temp(rhs); // 사본은 만든다.
    swap(temp);       // 사본과 맞바꾼다.
    return *this;
}
```