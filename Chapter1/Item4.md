# 항목 4. 객체를 사용하기 전에 반드시 그 객체를 초기화하자
## 객체의 초기화
- 모든 객체는 **사용전에 초기화** 하는게 좋다.
```cpp
int x = 0; // int 직접 초기화
const char * text = "HELLO"; // 포인터 직접 초기화
double d;
std::cin >> d; // 입력 스트림에서 읽음으로써 초기화
```
- 생성자에서 지킬 규칙은 `"객체의 모든것을 초기화 하자"`이다.
- **대입**을 **초기화**와 헷갈리지 않는 것이 중요하다.
    - 생성자에서 **대입문**을 사용하지 않고 **이니셜라이저**를 사용한다.

```cpp
class A{
public:
    int x;
    int y;

    // 초기화를 사용한 생성자
    A(int theX, int theY)
    {
        x = theX;
        y = theY;
    }

    // 이니셜라이저를 사용한 생성자
    A(int theX, int theY) : x(theX), y(theY) {}
};
```

## 객체를 구성하는 데이터의 초기화 순서
1. **기본 클래스**는 **파생 클래스**보다 먼저 초기화된다.
2. 클래스 멤버는 **선언된 순서대로 초기화** 된다.

### 정적 객체(static object)
- 자신이 생성된 시점부터 프로그램이 끝날 때까지 살아 있는 객체이다.
    1. 전역 객체
    2. 네임스페이스 유효범위에서 정의된 객체
    3. 클래스 안에서 `static`으로 선언된 객체
    4. 함수 안에서 `static`으로 선언된 객체
    5. 파일 유효범위에서 `static`으로 정의된 객체
- 함수 안에 있는 정적 객체는 **지역 정적 객체**라고 한다.
- 나머지는 **비지역 정적 객체**라고 한다.
- 정적 객체는 **프로그램이 끝날 때 자동으로 소멸**된다.

## 번역 단위
- 컴파일을 통해 하나의 **목적 파일**을 만드는 바탕이 되는 소스 코드
- 번역은 소스의 언어를 기계어로 옮긴다는 의미이다.

### 별개의 번역 단위에서 정의된 비지역 정적 객체들의 초기화 순서는 정해져 있지 않다.
```cpp
class FileSystem
{
public:
    std::size_t numDisks() const;
};
extern FileSystem tfs;

class Directory
{
public:
    Directory::Directory(params)
    {
        std::size_t disks = tfs.numDisks();
    }
};

Directory tempDir(params);
```
- `tfs`가 `tempDir`보다 먼저 초기화 되지 않으면 `tempDir`의 생성자는 `tfs`가 초기화 되지도 않았는데 `tfs`를 사용하려고 할것이다.
- 서로 다른 단위에 전의된 비지역 정적 객체들 사이의 **상대적인 초기화 순서**는 정해져 있지 않다.
- 여러 단위에 있는 비지역 정적 객체들의 초기화 순서문제는 피해서 설계해야한다. (비지역 정적 객체를 지역 정적 객체로 바꾸는것을 통해)
- 설계에 약간의 변화만 주면 문제를 봉쇄할 수 있다.
  - 비지역 정적 객체를 하나씩 맡는 함수를 준비하고 이 안에 각 객체를 넣는 것이다.
```cpp
class FileSystem {...};   // 이전과 동일

FileSystem& tfs()         // tfs 객체를 이 함수로 대신한다. 
{                         // 이 함수는 클래스 안에 정적 멤버로 들어가도 된다.
    static FileSystem fs; // 지역 정적 객체를 정의하고 초기화한다.
    return fs;            // 이 객체에 대한 참조자를 반환한다.
}

class Directory {...};    // 이전과 동일

Directory::Directory(params) // 이전과 동일하다. tfs의 참조자였던 것이 tfs()로 바뀌었다는 것만 다르다.
{
    ...
    std::size_t disks = tfs().numDisks();
    ...
}

Directory& tempDir()      // tempDir 객체를 이 함수로 대신한다.
{                         // 이 함수는 Directory 클래스의 정적 멤버로 들어가도 된다.
    static Directory td;  // 지역 정적 객체를 정의/초기화한다.
    return td;            // 이 객체에 대한 참조자를 반환한다.
}
```