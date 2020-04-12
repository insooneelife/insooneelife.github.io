---
title: "[Unreal] Physics"
categories:
  - Markup
---

### Example

```c++
#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "Components/SkeletalMeshComponent.h"
#include "Animation/AnimInstance.h"
#include "Animation/AnimSequence.h"
#include "TestPhysicsGameMode.generated.h"

UCLASS(minimalapi)
class ATestPhysicsGameMode : public AGameModeBase
{
	GENERATED_BODY()

public:
	const static FName ModifiedBoneLists[];
	const static int ModifiedNumBones;

public:
	ATestPhysicsGameMode();

	virtual void StartPlay() override;

	FTransform GetAnimBoneTransform(FName bname, float time);

private:
	UAnimSequence * _idleAnim;
	USkeleton* _skeleton;
};
```

### Referencing animation bone transform

```c++
FTransform ATestPhysicsGameMode::GetAnimBoneTransform(FName bname, float time)
{
	const FReferenceSkeleton& refSkel = _skeleton->GetReferenceSkeleton();

	time = fmod(time, ((_idleAnim->GetNumberOfFrames() - 1) / _idleAnim->GetFrameRate()));
	if (time < 0) time = time + (_idleAnim->GetNumberOfFrames() - 1) / _idleAnim->GetFrameRate();
	
	FTransform res;
	res.SetIdentity();
	int32 boneIdx = refSkel.FindBoneIndex(bname);
	while (true) 
	{
		if (!refSkel.IsValidIndex(boneIdx))
		{
			break;
		}
		FTransform tf;
		_idleAnim->GetBoneTransform(tf, boneIdx, time, true);
		res = res * tf;

		boneIdx = refSkel.GetParentIndex(boneIdx);
	}
	return res;
}
```
