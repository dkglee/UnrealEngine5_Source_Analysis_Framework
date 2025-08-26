```cpp
class UWorld
{
	TObjectPtr<class ULineBatchComponent> LineBatcher;
}
```
---
```cpp
class USceneComponent : public UActorComponent

class UPrimitiveComponent : public USceneComponent

class ULineBatchComponent : public UPrimitiveComponent
```
---
- `ULineBatchComponent`는 `UActorComponent`인 것을 볼 수 있다.
	  
- `UActorComponent`의 경우 `Actor`에 사용되는 역할로써 만들어진 클래스인데 `UObject`인 `UWorld`의 멤버 변수로써 선언된 것을 볼 수 있다.
	  
	- 후일 렌더링을 언급하면서 자연스럽게 다루게 됨
	  
- 이 경우 `CreateDefaultSubObject`가 아닌 `NewObject`를 통해 동적 생성 함으로써 계층 구조를 만들게 된다.
	  
- 이전의 `Actor`의 생성 과정([[KeyWord (AActor 초기화 순서)]])을 보면 알다시피 `UActorComponent`를 `Scene`에 `Register`(등록) 시키는 과정이 있다.
	  
- 하지만 `NewObject`한 객체의 경우 위의 초기화 단계를 거치지 못하고 또한, `ULineBatchComponent`의 경우 `UWorld`, 즉, `UObject`에서 생성된 객체임으로 수동으로 `Register` 과정을 거쳐주어야 한다.

```cpp
// - ULineBatchComponent는 UPrimitiveComponent를 상속한다
// - ULineBatchComponent는 UActorComponent로 볼 수 있다
// - UWorld는 AActor가 아니지만 LineBatcher 용 UActorComponent(동적 객체)를 가진다
// - LineBatcher는 별도로 등록해야 한다
if (!LineBatcher)
{
	LineBatcher = NewObject<ULineBatchComponent>();
	LineBatcher->bCalculateAccurateBounds = false;
}

if (!LineBatcher->IsRegistered())
{
    // UActorComponent::RegisterComponentWithWorld 참조 (goto 39)
    LineBatcher->RegisterComponentWithWorld(this, Context);
}
```
- 이 경우 `SubObject`라는 표현보다는 그냥 `NewObject`로 동적 할당된 객체라고 표현하는 경우가 일반적이다.