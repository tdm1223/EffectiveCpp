# 항목 22. 데이터 멤버가 선언될 곳은 private 영역임을 명심하자
## 데이터 멤버가 public이면 안되는 이유
### 1. [문법적 일관성](/Chapter4/Item18.md)
- 데이터 멤버에 대해 접근할 수 있는 경우가 함수 뿐이라면 괄호를 붙여야 하는지 말아야 하는지 고려할 필요가 없다.

### 2. 접근성에 대해 정교한 제어
- 접근 불가, 읽기 전용, 읽기 쓰기 접근을 직접 구현할 수 있다.

```cpp
class AccessLevels{
public:
    int getReadOnly() const { return readOnly; }
    
    void setReadWrite(int value) { readWrite = value; }
    int getReadWrite() const { return readWrite; }

    void setWriteOnly(int value) { writeOnly = value; }
private:
    int noAccess;  // 접근 불가
    int readOnly;  // 읽기 전용 접근
    int readWrite; // 읽기 쓰기 접근
    int writeOnly; // 쓰기 전용 접근
}
```

### 3. 캡슐화
- 데이터 멤버를 계산식으로 대체할수 있다.
- 사용자는 절대로 클래스를 넘볼 수 없다.

## 캡슐화
- 데이터 멤버를 함수 인터페이스 뒤에 감추게 되면 구현상의 융통성을 전부 누릴 수 있다.
    - 데이터 멤버를 읽거나 쓸 때 다른 객체에 알림 메시지 보내기
    - 스레딩 환경에서 동기화 걸기
- 캡슐화를 진행하면 클래스의 불변속성을 유지하게 된다.
- `protected`는 `public`보다 더 많이 보호받고 있는것이 절대로 아니다.
- **캡슐화 관점**에서 쓸모있는 접근 수준은 `private`(캡슐화 제공)와 `private가 아닌 나머지`(캡슐화 없음) 둘뿐이다.
