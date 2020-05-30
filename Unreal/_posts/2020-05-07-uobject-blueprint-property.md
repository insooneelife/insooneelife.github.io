---
title: "[Unreal] UObject Blueprint Property"
categories:
  - UObject
  - Property
  - Blueprint
---


### UObject blueprint property
언리얼에는 UPROPERTY를 활용한 코드들이 많다.
하지만 그렇기 때문에 오히려 이 프로퍼티들을 잘못 사용해서 생기는 문제들도 많다.
(Collision 이벤트 콜백으로 주는 함수가 UFUNCTION이 아닌경우 정상동작하지 않는것도 같은 이유이다.)

UObject를 상속받은 UMyObject를 cpp 클래스로 생성한다.
UMyObject를 사용한 blueprint 클래스를 생성하려 하면 에디터 상에서 표시되지 않는 문제가 있다.
이는 UCLASS에 에디터에 표시되도록 하기 위한 파라메터를 적절하게 주지 않아서 발생하는 문제이다.

```c++
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "MyObject.generated.h"

// blueprint에서 사용가능하도록 하기 위함
UCLASS(Blueprintable, BlueprintType)
class FACEFXEXAMPLE_API UMyObject : public UObject
{
	GENERATED_BODY()
	
};
```

이제 UMyObject를 사용한 blueprint 클래스를 생성할 수 있다.



