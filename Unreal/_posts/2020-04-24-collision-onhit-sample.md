---
title: "[Unreal] Collision Event OnHit"
categories:
  - Collision
---

### Collision OnHit Event
Unreal engine 충돌 관련 예제이다.


```c++

void MyActor::BeginPlay()
{
...
  //set hit delegate
	FScriptDelegate hitDelegate;
	hitDelegate.BindUFunction(this, TEXT("OnHit"));
	this->GetAttachmentRootActor()->OnActorHit.AddUnique(hitDelegate);
}

// OnHit event 함수는 반드시 UPROPERTY로 정의되어야 한다.
// 호출을 위해서는 현재 Actor와 대상 Actor의 Collistion Preset이 Block으로 지정되어 있어야 한다.
void MyActor::OnHit(AActor* SelfActor, AActor* OtherActor, FVector NormalImpulse, const FHitResult& Hit) 
{
	
	FVector HitWorldLocation;
	FVector HitBoneLocation;
	FRotator HitBoneRotation;

	HitWorldLocation = Hit.Location;
	//FName boneName = Hit.BoneName;
	FName boneName = Hit.MyBoneName;
	FRotator HitWorldRotation = SkeletalMeshComponent->GetSocketRotation(boneName);
	SkeletalMeshComponent->TransformToBoneSpace(boneName, HitWorldLocation, HitWorldRotation, HitBoneLocation, HitBoneRotation);
	
	if (JointMap.Contains(boneName.ToString()))
	{
		int JointInd = JointMap[boneName.ToString()];
		UE_LOG(LogHolodeck, Warning, TEXT(
			"OnHit !!!!!!!!!!!!!  name : %s  index : %d  this actor : %s  other actor : %s"),
			*boneName.ToString(), JointInd, *SelfActor->GetFName().ToString(), *OtherActor->GetFName().ToString());
	}

}

```

#### 실행 결과



