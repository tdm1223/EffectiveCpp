# 항목 20. '값에 의한 전달' 보다는 '상수객체 참조자에 의한 전달' 방식을 택하는 편이 대개 낫다
### 값에 의한 전달의 비용
```cpp
class Person{
public:
    std::string name;
    std::string address;
};

class Student : public Person{
public:
    std::string schoolName;
    std::string schoolAddress;
};

bool validateStudent(Student s);
bool validateStudent(const Student& s);
```
- 첫번째 `validateStudent` 함수에 `Student` 객체 전달시의 비용
    - 생성자 **여섯 번**(Person, Student, string x 4)
    - 소멸자 **여섯 번**(Person, Student, string x 4)
- 함수에 클래스를 전달할때 두번째 `validateStudent` 함수처럼 **상수 객체 참조자**를 넘기는것이 좋다.

### 복사 손실 문제
```cpp
class Window{
public:
    std::string name() const;
    virtual void display() const;
};

class WindowWithScrollBars : public Window{
public:
    virtual void display() const;
};

void printNameAndDisplay(Window w) // 복사 손실을 당하는 매개변수
{
    std::cout<<w.name();
    w.display(); 
}

void printNameAndDisplay(const Window& w) // 복사 손실을 방지하기 위해 상수객체 참조자 전달
{
    std::cout<<w.name();
    w.display();
}
WindowWithScrollBars wwsb;
printNameAndDisplay(wwsb);
```
- 매개변수 `w`가 생성되기는 하지만 `Window` 객체로 만들어지면서 `wwsb`가 `WindowWithScrollBars` 객체의 구실을 할 수 있는 부속 정보가 모두 없어진다.
    - **매개변수**가 **복사 손실**을 당하게 된다.
- 참조에 의한 전달 방식으로 매개변수를 넘기면 **복사손실 문제**가 없어진다.
    - `void printNameAndDisplay(const Window& w);`
- **참조자를 전달**한다는 것은 **포인터를 전달**한다는 것과 일맥상통하다.

### 크기가 작으면 값에 의한 전달?
- 크기가 작다는 객체의 복사 생성자 호출이 저비용이란 뜻이 아니다.
- 크기가 작아도 **복사하는 비용**은 **고비용**일 수 있다. (포인터 멤버가 가리키는 대상까지 복사한다면?)
- 값에 의한 전달이 저비용이라고 가정해도 괜찮은 타입
    1. 기본제공 타입
    2. STL반복자
    3. 함수 객체타입
- 위 3가지 항목 외의 타입에 대해서는 `상수객체 참조자`에 의한 전달을 선택하는것이 좋다.