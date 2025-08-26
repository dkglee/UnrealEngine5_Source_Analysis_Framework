### Persistent Level
```cpp
class UWorld
{
	TObjectPtr<class ULevel> PersistentLevel;
}
```
- 이 월드가 사라지지 않는 한 `UnLoad`되지 않는 레벨을 의미함
	  
- 즉, Streaming Out 혹은 In이 되지 않는 레벨을 의미함.
	  
- 내부에 `WorldSetting`을 가지고 있음

```cpp
PersistentLevel->SetWorldSettings(WorldSettings);
```
### Streaming Level
- Persistent Level과 달리 Streaming 되는 레벨
	  
- 메모리 부하를 막기 위하여