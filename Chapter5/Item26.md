# 항목 26. 변수 정의는 늦출 수 있는 데까지 늦추는 근성을 발휘하자
- **생성자**나 **소멸자**를 끌고 다니는 타입으로 변수를 정의하면 물게 되는 비용은 두개이다.
    - 프로그램 제어 흐름이 변수의 정의에 닿을 때 생성자가 호출되는 비용
    - 변수가 유효범위를 벗어날 때 소멸자가 호출되는 비용
- 변수가 정의됐으나 사용되지 않은 경우에도 비용이 부과된다.
```cpp
std::string encryptPassword(const std::string& password)
{
    using namespace std;

    string encrypted; // string 변수 선언

    if (password.length() < MinimumPasswordLength)
    {
        throw logic_error("Password is too short");
    }

    // 비밀번호 암호화 하는 로직

    return encrypted;
}
```
- `encrypted` 객체는 **예외가 발생**하면 사용되지 않는 객체이다.
- 해당 변수를 정의하는 일은 **예외처리 아래**까지 미루는 편이 낫다.

### 조금더 미루어보자
```cpp
std::string encryptPassword(const std::string& password)
{
    if (password.length() < MinimumPasswordLength)
    {
        throw logic_error("Password is too short");
    }

    using namespace std;

    string encrypted; // 기본 생성자에 의해 만들어지는 encrypted
    
    encrypted = password; // encrypted에 password를 대입

    // 비밀번호 암호화 하는 로직

    return encrypted;
}
```
- 위 코드에서 바람직한 방법은 `encrypted`를 `password`로 초기화해 버리는 방법이다.
- 의미도 없고 비용도 만만치 않을 듯한 **기본 생성자** 호출을 건너뛸 수 있게 된다.

### 조금더 미루어보자 2
```cpp
std::string encryptPassword(const std::string& password)
{
    // 같은 로직

    string encrypted(password); // 정의와 동시에 초기화. 복사 생성자가 사용된다.

    encrypt(encrypted);
    return encrypted;
}
```
- **정의와 동시에 초기화** 함으로써 **성능**을 높일 수 있고 프로그램도 깔끔해 진다.
