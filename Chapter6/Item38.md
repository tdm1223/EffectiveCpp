# 항목 38. "has-a(...는...를 가짐)" 혹은 "is-implemented-in-terms-of(...는...를 써서 구현됨)"를 모형화할 때는 객체 합성을 사용하자
## 합성
- `Person` 객체는 `Address`, `PhoneNumber` 객체로 이루어져 있다.
```cpp
class Address{};

class PhoneNumber{};

class Person{
public:
    Address address;
    PhoneNumber phoneNumber;
};
```
- 어떤 타입의 객체들이 그와 다른 타입의 객체들을 포함하고 있을 경우 성립하는 타입들 사이의 관계를 합성이라 한다.
- 레이어링, 포함, 통합, 내장등으로도 불린다.

## 응용 영역과 구현 영역
### 응용 영역
- 일상생활에서 볼 수 있는 사물을 본뜬 객체이다.
- 예로는 사람, 이동수단, 비디오 프레임등이 있다.
- 객체 합성이 **응용 영역**의 객체들 사이에서 일어나면 `has-a` 관계이다.

### 구현 영역
- 버퍼, 뮤텍스, 탐색트리 등 순수하게 시스템 구현만을 위한 인공물
- 객체 합성이 **구현 영역**에서 일어나면 `is-implemented-in-terms-of` 관계이다.
- `A는 B을 써서 구현되는`형태의 설계를 한다면 `is-implemented-in-terms-of` 관계이다.
```cpp
template<class T>
class Set{
public:
    bool member(const T& item) const;
    void insert(const T& item);
    void remove(const T& item);
    std::size_t size() const;

private:
    std::list<T> rep;
}
```
- 자료구조 set을 자료구조 list를 이용하여 만들기 때문에 `is-implemented-in-terms-of` 관계라고 할 수 있다.