---
title: "[Unreal] Physics"
categories:
  - Markup
---

### Example

```c++
// MIT License (c) 2019 BYU PCCL see LICENSE file

#pragma once

#include "GameFramework/Pawn.h"
#include "Components/SkeletalMeshComponent.h"
#include "Android.generated.h"

UCLASS()
class HOLODECK_API AAndroid : public ACharacter
{
	GENERATED_BODY()

public:
	/**
	* Default Constructor
	*/
	AAndroid();

	UPROPERTY(BlueprintReadWrite, Category = AndroidMesh)
		USkeletalMeshComponent* SkeletalMesh;

};
```

### Left Align

This is a paragraph. It is left aligned. Because of this, it is a bit more liberal in it's views. It's favorite color is green. Left align tends to be more eco-friendly, but it provides no concrete evidence that it really is. Even though it likes share the wealth evenly, it leaves the equal distribution up to justified alignment.
{: style="text-align: left;"}
