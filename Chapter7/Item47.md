# 항목 47. 타입에 대한 정보가 필요하다면 특성정보 클래스를 사용하자
## STL advance
- 반복자를 **지정된 거리만큼 이동**시키는 템플릿
```cpp
template<typename IterT, typename DistT> // iter를 d 단위만큼 전진시킨다.
void advance(IterT& iter, DistT d);      // d < 0 이면 iter를 후진시킨다.
```

## STL 반복자
### 입력 반복자
- 전진만 가능
- 한번의 한칸씩만 이동
- 자신이 가리키는 위치에서 읽기만 가능
- 읽을수 있는 횟수 1번
- `istream_iterator`
- 단일 패스 알고리즘에만 제대로 사용 가능

### 출력 반복자
- 전진만 가능
- 한번의 한칸씩만 이동
- 자신이 가리키는 위치에서 쓰기만 가능
- 쓸 수 있는 횟수 1번
- `ostream_iterator`
- 단일 패스 알고리즘에만 제대로 사용 가능

### 순방향 반복자
- 입력 반복자와 출력 반복자가 하는 일은 다 할수 있다.
- 자신이 가리키고 있는 위치에서 읽기와 쓰기 동시에 가능
- 여러번 읽고 쓰기 가능
- 다중 패스 알고리즘에 문제 없이 사용 가능

### 양방향 반복자
- 순방향 반복자에 뒤로 갈 수 있는 기능을 추가한것
- `STL`의 `list`에 쓰는 반복자가 이 범주에 들어간다.
- `set`, `multiset`, `map`, `multimap` 등의 컨테이너에도 양방향 반복자를 사용

### 임의 접근 반복자
- 양방향 반복자에 반복자 산술 연산 수행 기능을 추가
- `vector`, `deque`, `string`에 사용하는 반복자

### 태그 구조체
- `C++` 표준 라이브러리에는 다섯 개의 반복자 범주 각각을 식별하는데 쓰이는 **태그 구조체**가 정의되어 있다.
  - `struct input_iterator_tag {};`
  - `struct output_iterator_tag {};`
  - `struct forward_iterator_tag : publi input_iterator_tag {};`
  - `struct bidirectional_iterator_tag : public forward_iterator_tag {};`
  - `struct random_access_iterator_tag : public bidirectional_iterator_tag {};`

## 특성정보
- **컴파일 도중**에 어떤 주어진 타입의 정보를 얻을 수 있게 하는 객체를 지칭하는 개념
```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
    if(iter가 임의 접근 반복자) // 특성정보를 통해 임의 접근 반복자인지 판별할 수 있다.
    {
        iter += d;
    }
    else
    {
        if(d >= 0) { while(d--) ++iter; }
        else{ while(d++) --iter; }
    }
}
```
- 코드가 제대로 되려면 `iter` 부분이 **임의 접근 반복자**인지 판단할 수 있는 수단이 있어야 한다.
- 특성정보는 정의된 문법구조나 키워드가 아니고 관례이다.
  - 특성정보는 **기본제공 타입**과 **사용자 정의 타입**에서 모두 돌아가야 한다.
- 특성정보를 템플릿 및 그 템플릿의 1개 이상의 특수화 버전에 넣는다.
- 반복자의 경우 표준 라이브러리의 특성정보용 템플릿이 `iterator_traits`라는 이름으로 준비되어 있다.
  - 포인터 타입의 반복자를 지원하기 위해 **부분 템플릿 특수화 버전**도 제공한다.

## 특성정보 클래스
- 특성 정보를 구현하는 데 사용한 구조체

### 특성 정보 클래스의 구현
- 다른 사람이 사용하도록 열어 주고 싶은 **타입 관련 정보**를 확인한다.(반복자라면 반복자 범주 등..)
- 정보를 식별하기 위한 **이름**을 선택한다.
- 지원하고자 하는 타입 관련 정보를 담은 **템플릿** 및 그 템플릿의 **특수화 버전**을 제공한다.
```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
    if (typeid(typename std::iterator_traits<IterT>::iterator_category) == typeid(std::random_access_iterator_tag))
    {
        ...
    }
}
```
- 컴파일이 실패한다.
  - `IterT`의 타입은 **컴파일 도중에 파악** 되기 때문에 `iterator_traits<IterT>::iterator_category`를 파악할 수 있는 것도 컴파일 도중이다.
  - `if` 문은 프로그램 실행 도중에 평가된다.

### 오버로딩을 통한 해결
- `advance`의 동작 원리 알맹이는 같게한다.
- 받아들이는 `iterator_category` 객체의 타입을 다르게 해서 오버로드 함수를 만든다.

```cpp
 // 임의 접근 반복자에 대해서는 이 구현을 사용
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::random_access_iterator_tag)
{
    iter += d;
}

// 양방향 반복자에 대해서는 이 구현을 사용
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::bidirectional_iterator_tag)
{
    if (d >= 0) { while (d--)++iter; }
    else { while (d++)--iter; }
}

// 입력 반복자에 대해서는 이 구현을 사용
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::input_iterator_tag)
{
    if (d < 0)
    {
        throw std::out_of_range("Negative distance");
    }
    while(d--) ++iter;
}

// iter의 반복자 범주에 적합한 doAdvance의 오버로드 버전 호출
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
    doAdvance(iter, d, typename std::iterator_traits<IterT>::iterator_category());
}
```
- **작업자 역할**을 맡을 함수 혹은 함수 템플릿
    - 특성정보 매개변수를 다르게 하여 **오버로딩** 한다.
    - 전달되는 해당 특성정보에 맞추어 각 오버로드 버전을 구현한다.
- 작업자를 호출하는 **주작업자 역할**을 맡을 함수 혹은 함수 템플릿
    - 특성정보 클래스에서 제공되는 정보를 넘겨서 **작업자를 호출**하도록 구현한다.