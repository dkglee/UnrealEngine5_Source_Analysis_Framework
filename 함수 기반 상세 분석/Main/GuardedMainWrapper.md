```cpp
int32 GuardedMainWrapper(const TCHAR* CmdLine)
{
    int32 ErrorLevel = 0;
    // see GuardMain()
    // deulee :
    // - 즉 여기가 진짜 Main의 시작점이다.
    ErrorLevel = GuardMain(CmdLine);
    return ErrorLevel;
}
```
---
SEH를 사용하여 감싸져 있는 모습