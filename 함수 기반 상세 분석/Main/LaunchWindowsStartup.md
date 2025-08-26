---
tags:
  - Main
---
```cpp
int32 LaunchWindowsStartup(HINSTANCE hInInstance, HINSTANCE hPrevInstance, char*, int32 nCmdShow, const TCHAR* CmdLine)
{
    int32 ErrorLevel = 0;

	// Process Command Line을 통해 언리얼에서 사용 가능한 CMD로 변환
    if (!CmdLine)
    {
        CmdLine = GetCommandLineW();

        if (ProcessCommandLine())
        {
            CmdLine = *GSavedCommandLine;
        }
    }

    {
	    // *SEH* 구조로 처리하는 것을 볼 수 있음
	    // 
        __try
        {
            GIsGuarded = 1;
            // see GuardedMainWrapper()
            // 실질적으로 실행하는 부분을 감싼것을 볼 수 있음.
            // SEH로 인하여 에러가 발생하면 except 문으로 들어감
            ErrorLevel = GuardedMainWrapper(CmdLine);
            GIsGuarded = 0;
        }
        __except(FPlatformMisc::GetCrashHandlingType == ECrashHandlingType::Default
            ? ReportCrash(GetExceptionInformation())
			: EXCEPTION_CONTINUE_SEARCH)
        {
            ErrorLevel = 1;
            FPlatformMisc::RequestExit(true);
        }
    }

    return ErrorLevel;
}
```
---
1. Command Line을 나름대로 파싱하고 재조립하여 언리얼에서 사용할 수 있게끔 함
	- 예시 : `-tracehost=127.0.0.1 -trace=cpu,gpu,frame`