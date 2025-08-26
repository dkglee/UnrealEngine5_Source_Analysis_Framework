```cpp
/** specifies the goal/source of a UWorld object */
namespace EWorldType
{
	enum Type
	{
		/** An untyped world, in most cases this will be the vestigial worlds of streamed in sub-levels */
		None,

		/** The game world */
		Game,

		/** A world being edited in the editor */
		Editor,

		/** A Play In Editor world */
		PIE,

		/** A preview world for an editor tool */
		EditorPreview,

		/** A preview world for a game */
		GamePreview,

		/** A minimal RPC world for a game */
		GameRPC,

		/** An editor world that was loaded but not currently being edited in the level editor */
		Inactive
	};
}
```
- `UWorld` 객체의 목표 및 어떤 종류인지 나타내는 `enum`
- 언리얼에서 `enum`을 정의할 때 굉장히 많이 사용되는 패턴
- `namespace`로 래핑함으로써 다른 `global enum`과의 충돌을 피할 수 있음
	
	- 이러면 C++에서만 사용 가능한 enum 일듯? `Reflection`이 안되니까?

