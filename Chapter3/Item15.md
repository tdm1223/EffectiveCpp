# 항목 15. 자원 관리 클래스에서 관리되는 자원은 외부에서 접근할 수 있도록 하자
- 자원관리 클래스는 매우 훌륭하므로 이 클래스만 사용하면 될것이다.
- 그러나 현장에서 쓰이는 많은 API들이 자원을 직접 참조하도록 만들어져 있다.
- 실제 자원을 직접 접근할 일이 생기게 된다.

## RAII 클래스의 객체를 객체가 감싸고 있는 실제 자원으로 변환할 방법이 필요할때
- 일반적인 방법으로는 아래 두가지 방법이 있다.
    1. 명시적 변환 : 안정성
    2. 암시적 변환 : 고객 편의성

- 명시적 변환을 제공할 것인지 암시적 변환을 허용할 것인지에 대한 결정은 그 `RAII` 클래스 만의 특정한 용도와 사용 환경에 따라 달라진다.
- 가장 잘 설계한 클래스는 [맞게 쓰기에는 쉽게, 틀리게 쓰기에는 어렵게](/Chapter4/Item18.md) 만들어져야 한다.

### 명시적 변환
- `shared_ptr` 및 `auto_ptr`은 **명시적 변환**을 수행하는 `get`이라는 멤버 함수를 제공한다.
- `get`함수를 통해 스마트 포인터 객체에 들어있는 실제 포인터를 얻어낼 수 있다.
```cpp
int count = countHeld(countPtr).get());
```

### 암시적 변환
- 스마트 포인터 클래스는 포인터 역참조 연산자(`operator->` 및 `operator*`)도 오버로딩 하고 있다.
- 이 연산자를 통해 자신이 관리하는 실제 포인터에 대한 암시적 변환도 쉽게 할 수 있다.
```cpp
class TestClass {
public:
    bool Func() const;
};

TestClass* createTestClass(); // 팩토리 함수

std::shared_ptr<TestClass> ptr1(createTestClass());

bool taxable1 = !(ptr1->Func()); // operator->를 써서 자원에 접근

std::shared_ptr<TestClass> ptr2(createTestClass());

bool taxable2 = !((*ptr2).Func()); // operator*를 써서 자원에 접근
```

## RAII와 캡슐화
- `RAII` 클래스는 애초부터 데이터 은닉이 목적이 아니다.
- 자원 해제가 실수 없이 이루어 지면 된다.
- `shared_ptr`의 경우도 **참조 카운팅 메커니즘**에 필요한 장치들은 모두 캡슐화 하고 있지만, **자신이 관리**하는 포인터를 쉽게 접근할 수 있는 통로도 제공한다.

