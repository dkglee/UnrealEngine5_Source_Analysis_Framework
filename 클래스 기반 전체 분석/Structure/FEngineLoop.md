```cpp
FEngineLoop GEngineLoop; // 싱글톤 객체

class FEngineLoop : public IEngineLoop
{
public:
    /** pre-initialize the main loop - parse command line, sets up GIsEditor, etc */
    int32 PreInit(const TCHAR* CmdLine);
	virtual int32 Init() override;
    virtual void Tick() override;
};
```

---

- 실질적으로 엔진의 `PreInit`, `Init`, `Tick`을 담당하는 **싱글톤** 객체
- `UEngine`을 생성함
