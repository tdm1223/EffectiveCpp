# 항목 44. 매개변수에 독립적인 코드는 템플릿으로부터 분리시키자
### 템플릿을 포함한 프로그램이 코드 비대화를 일으키는 상황
- 고정 크기의 정방행렬을 나타내는 클래스 템플릿
```cpp
template<typename T, std::size_t n>
class SquareMatrix{
public:
    void invert(); // 주어진 행렬을 저장 공간에서 역행렬로 만든다.
};

SquareMatrix<double, 5> sm1; // SquareMatrix<double, 5>::invert 호출
sm1.invert();

SquareMatrix<double, 10> sm2; // SquareMatrix<double, 10>::invert 호출
sm2.invert();
```
- 위 템플릿은 `T`라는 **타입 매개변수**도 받지만, `size_t` 타입의 **비타입 매개변수**인 `n`도 받는다.
- 함수 `invert`의 사본이 인스턴스화 되는데, 만들어지는 사본의 개수가 두 개이다.
    - 둘은 같은 함수일 수가 없다.
    - 행과 열 크기를 나타내는 상수만 빼면 두 함수는 완전히 같다.
- 템플릿으로 인해 코드 비대화가 일어난다.

### 코드 비대화를 줄이기 위한 노력
```cpp
template<typename T>
class SquareMatrixBase{
protected:
    void invert(std::size_t matrixSize); // 주어진 크기의 행렬을 역행렬로 만든다.
};

template<typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T>{
private:
    using SquareMatrixBase<T>::inmvert;
public:
    void invert() {this->invert(n);} // invert의 기본 클래스 버전에 대해 인라인 호출 수행
};
```
- 같은 타입의 객체를 원소로 갖는 모든 정방행렬은 오직 한 가지의 `SquareMatrixBase` 클래스를 공유한다.
    - 기본 클래스 버전의 `invert` 함수도 오직 **한개의 사본**이다.
- `SquareMatrixBase::invert` 함수는 파생 클래스에서 **코드 복제를 피할 목적으로만 마련한 장치**이기 때문에, `protected` 멤버로 선언한다.
  - 함수 호출에 추가 비용이 없어야 한다.(this를 쓰는 이유)

### 메모리 할당 방법의 결정 권한을 파생 클래스로 넘기는 방법
```cpp
template<typename T>
class SquareMatrixBase{
protected:
    SquareMatrixBase(std::size_t m, T *pMem) : size(n), pData(pMem){} // 행렬 크기를 저장하고 행렬 값에 대한 포인터를 저장
    void setDataPtr(T * ptr) {pData = ptr;} // pData에 다시 대입
private:
    std::size_t size; // 행렬의 크기
    T *pData; // 행렬 값에 대한 포인터
};

template<typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T>{
public:
    SquareMatrix() : SquareMatrixBase<T>(n, data) {} // 행렬의 크기 및 데이터 포인터를 기본 클래스로 올려 보낸다.
private:
    T data[n*n];
}
```
- 행렬 값을 담을 **메모리 할당 방법의 결정 권한**이 **파생 클래스** 쪽으로 넘어간다.
- 동적 메모리 할당이 필요 없는 **파생 클래스**가 되지만, 객체 자체의 크기가 커질 수 있다.
    - 행렬의 데이터를 힙에 두면 된다.

### 데이터를 힙에 두는 방법
```cpp
template<typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T>{
public:
    SquareMatrix() : SquareMatrixBase<T>(n, 0), pData(new T[n*n]) { this->setDataPtr(pdData.get());}
private:
    boost::scoped_arrayt<T> pData;
};
```
- `SquareMatrix`에 속해 있는 멤버 함수 중 상당수가 **기본 클래스 버전**을 호출하는 **단순 인라인 함수**가 될 수 있다.
- 똑같은 타입의 데이터를 원소로 갖는 모든 정방행렬들이 행렬 크기에 상관없이 **기본 클래스 버전의 사본 하나**를 공유한다.
- 행렬 크기가 다른 `SquareMatrix` 객체는 저마다 고유의 타입을 갖고 있다.
    - `SquareMatrix<double, 5>`, `SquareMatrix<double, 10>`객체가 `SquareMatrixBase<double>` 클래스의 멤버 함수를 사용하고 있다 하더라도 둘은 타입이 다르기 때문에 다른 타입의 함수 호출을 컴파일러가 막아준다.
- 크기별 고정 버전의 경우 행렬 크기가 컴파일 시점에 투입되는 상수이기 때문에 상수 전파등의 최적화 하기에 좋다.

- 행렬 크기에 대해 한 가지 버전의 `invert`를 두도록 만들면 **실행 코드의 크기**가 작아진다.
  - 프로그램의 작업 세트 크기가 줄어들면서 명령어 캐시 내의 참조 지역성도 향상된다.
  - 프로그램 실행 속도가 더 빨라질 수 있다.