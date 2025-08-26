```cpp
/**
 * 모든 UE 객체들의 기반 클래스입니다. 객체의 타입은 UClass에 의해 정의됩니다.
 * 객체를 생성하고 사용하는 데 필요한 지원 함수들과,
 * 자식 클래스에서 오버라이드해야 하는 가상 함수들을 제공합니다.
 */
class UObject : public UObjectBaseUtility
{
    
};
```
---
# UObject

## 1) 클래스의 역할/책임
- 모든 UE 객체들의 기반 클래스입니다. 객체의 타입은 UClass에 의해 정의됩니다.
- 객체를 생성하고 사용하는 데 필요한 지원 함수들과, 자식 클래스에서 오버라이드해야 하는 가상 함수들을 제공합니다.

## 2) 클래스가 제공하는 기능


## 3) 관계
### 3.1 직접 상속(IsA)
> 대표적인 **직접 상속자(일부 예시)**
- `UField`
- `UPackage`
- `UWorld`
- `UEngine`
- `UActorComponent`

### 3.2 포함(Has)


## 4) 상속 계보(참고)
- Base Classes:
  - `UObjectBaseUtility` ← `UObject`

## 5) 멤버 변수(이 클래스가 포함하고 있는 변수)


### 참고
[[UObjectBaseUtility]]
