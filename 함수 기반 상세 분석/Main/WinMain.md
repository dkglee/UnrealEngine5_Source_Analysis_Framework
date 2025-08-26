---
tags:
  - Main
---
```cpp
int32 WinMain(_In_ HINSTANCE hInInstance, _In_opt_ HINSTANCE hPrevInstance, _In_ char* pCmdLine, _In_ int32 nCmdShow)

{
    // see LaunchWindowsShutdown()
    // see LaunchWindowsStartup()
    int32 Result = LaunchWindowsStartup(hInInstance, hPrevInstance, pCmdLine, nCmdShow, nullptr);
    LaunchWindowsShutdown();
    return Result;
}
```
---
Launch Windows Startup() 함수에서 엔진의 모든 초기화, 루프 동작이 있는 것을 알 수 있다.