# 항목 8. 예외가 소멸자를 떠나지 못하도록 붙들어 놓자
## 소멸자와 예외
- **소멸자**에서 예외가 나가도록 내버려두면 안된다.
- **소멸자**가 예외를 발생하지 않도록 해야 한다.

## 소멸자가 예외를 던지는걸 막는 방법
- 아래와 같이 데이터베이스 연결을 나타내는 클래스가 있다.
- 사용자가` DBConnection` 객체에 대해 `close`를 직접 호출해야 하는 상태이다.
- 자원 관리 클래스(`DBConn`)을 생성하여 `close`의 망각을 사전에 차단할 수 있다.
```cpp
class DBConnection{
public:
    static DBConnection create(); // DBConnection 객체를 반환하는 함수

    void close(); // 연결응 닫는다. 연결이 실패하면 예외를 던진다.
};

class DBConn{
public:
    ~DBConn()
    {
        db.close();   // DB 연결이 항상 닫히도록 챙겨주는 함수
    }
private:
    DBConnection db;
}
```
- `close`에서 예외가 발생하면 `DBConn`의 소멸자는 예외를 전파할 것이다.
- 이런 문제를 피하는 방법은 두가지이다.

### 예외가 발생하면 프로그램을 바로 끝낸다.
- `abort()`를 호출한다.
```cpp
DBConn::~DBConn()
{
    try{ db.close(); }
    catch (...){
        // close 호출이 실패했다는 로그 작성
        std::abort();
    }
}
```

### 발생한 예외를 삼켜버린다.
- 예외 삼키기는 대부분의 경우 좋은 방법은 아니다.

```cpp
class DBConn {
public:
    void close() {
        db.close();
        closed = true;
    }

    ~DBConn() {
        if (!closed) {
            try {
                db.close(); // 사용자가 연결을 안닫았으면 닫아본다.
            }
            catch (...) {
                // close 호출이 실패했다는 로그를 작성
            }
        }
    }
private:
    DBConnection db;
    bool closed;
};
```
- `close` 호출의 책임을 `DBConn`의 **소멸자**에서 `DBConn`의 **사용자**로 떠넘기는 방법이다.
- 예외를 일으키면서 실패할 가능성이 있고 예외를 처리해야 할 필요가 있다면 예외는 **소멸자가 아닌 다른 함수에서 비롯된 것**이어야 한다.