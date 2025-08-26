```cpp
/** preinit the engine loop */
int32 EnginePreInit(const TCHAR* CmdLine)
{
    int32 ErrorLevel = GEngineLoop.PreInit(CmdLine);
    return ErrorLevel;
}
```
---
`GEngineLoop`의 `PreInit` 함수를 호출하여 엔진의 사전 초기화를 진행한다.