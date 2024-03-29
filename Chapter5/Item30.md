# 항목 30. 인라인 함수는 미주알고주알 따져서 이해해 두자
## 인라인 함수
- 함수처럼 보이고 함수처럼 동작한다.
- [매크로보다 훨씬 안전](/Chapter1/Item2.md)하고 쓰기 좋다.
- 함수 호출 시 발생하는 오버헤드를 걱정할 필요가 없다.
- 인라인 함수를 사용하면 컴파일러가 함수 본문에 문맥별 최적화를 걸기가 용이하다.
- 본문 길이가 짧은 인라인 함수를 사용하면, **함수 본문**에 대해 만들어지는 코드의 크기가 **함수 호출문**에 대해 만들어지는 코드보다 작아질 수 있다.
    - 목적 코드의 크기가 **작아**진다.
    - 명령어 **캐시 적중률**이 높아진다.
- 인라인을 남발하면 프로그램 크기가 커질 수 있다.
    - 기계에서 쓸 수 있는 공간을 넘을 수 있다.
    - 가상 메모리를 쓰더라도 성능의 걸림돌이 될 수 있다.
    - 페이징 횟수가 늘어난다.
    - 명령어 캐시 적중률이 떨어진다.
- 인라인 함수는 대체적으로 **헤더 파일**에 들어 있어야 하는게 맞다.
    - 대부분의 빌드 환경에서 인라인을 컴파일 도중에 수행하기 때문이다.
- 어떤 템플릿으로부터 만들어지는 모든 함수가 인라인 함수였으면 싶은 경우 템플릿에 `inline`을 붙여 선언한다.
- 인라인 함수는 대부분의 디버거가 곤란해 하는 비호감 대상이다.
- 생성자와 소멸자는 인라인 하기 좋지 않은 함수이다.

## 인라인 함수의 후보 결정하기
- 컴파일러가 인라인 함수의 후보들을 결정한다.
- `inline`은 컴파일러 선에서 무시할 수 있는 **요청**이고 컴파일러가 판단하여 `inline` 함수로 바꾼다.
- 인라인 함수로 선언되어 있어도 컴파일러가 보기에 복잡한 함수는 인라인 확장 대상에 넣지 않는다.
- 간단한 함수여도 가상 함수 호출은 절대로 인라인해 주지 않는다.
### 클래스 정의 안에 함수를 바로 정의해 넣으면 컴파일러는 함수를 인라인 후보로 찍는다.
```cpp
class Person{
public:
    int age() const {return theAge;} // 암시적인 인라인 요청
private:
    int theAge;
}
```
### 함수 정의 앞에 inline 키워드를 붙인다.
- 표준 라이브러리의 `max` 템플릿이 있다.
```cpp
template<typename T>
inline const T& std::max(const T& a, const T& b)
{
    return a < b ? b : a;
}
```
## inline 사용 기본 전략
- 우선은 아무것도 인라인하지 않는다.
- 꼭 [인라인 해야 하는 단순한 함수](/Chapter7/Item46.md)에 한해서만 인라인 함수로 선언한다.
- 디버깅하고 싶은 부분에서 디버거를 제대로 쓸 수 있도록 만들어야 한다.
- 필요한 위치에 인라인 함수를 놓는다(수동 최적화)