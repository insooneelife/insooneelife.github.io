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

### Referencing animation bone transform

```c++
FTransform ATestPhysicsGameMode::GetAnimBoneTransform(FName b_name, float time)
{
	const FReferenceSkeleton& refSkel = skeleton->GetReferenceSkeleton();

	time = fmod(time, ((IdleAnim->GetNumberOfFrames() - 1) / IdleAnim->GetFrameRate()));
	if (time < 0) time = time + (IdleAnim->GetNumberOfFrames() - 1) / IdleAnim->GetFrameRate();
	
	//UE_LOG(LogTemp, Warning, TEXT("time: %f"), time);
	FTransform res;
	res.SetIdentity();
	FName cur_body_name = b_name;
	int32 boneIdx = refSkel.FindBoneIndex(cur_body_name);
	while (true) 
	{
		if (!refSkel.IsValidIndex(boneIdx))
		{
			break;
		}
		FTransform tf;
		IdleAnim->GetBoneTransform(tf, boneIdx, time, true);
		res = res * tf;

		boneIdx = refSkel.GetParentIndex(boneIdx);
	}
	return res;
}
```
