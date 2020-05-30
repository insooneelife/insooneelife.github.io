---
title: "[Unreal] Collision By Animation Notify"
categories:
  - Animation
  - Collision
---


### Collision by animation notify
애니메이션 notify를 통해 캐릭터 객체의 공격 궤적에 따른 충돌처리 예제이다.



#### header
```c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "MyCharacter.generated.h"

class UStaticMeshComponent;

UCLASS()
class ANIMATIONRETARGET_API AMyCharacter : public ACharacter
{
	GENERATED_BODY()

public:
	// Sets default values for this character's properties
	AMyCharacter();

	UFUNCTION(BlueprintCallable, Category = "AMyCharacter")
		void AttackStart();

	UFUNCTION(BlueprintCallable, Category = "AMyCharacter")
		void AttackEnd();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;
	 
private:
	FBodyInstance * RHand;	
	FVector SaveAttackStartPos;
};

```


#### source
```c++
void AMyCharacter::BeginPlay()
{
	Super::BeginPlay();
	this->RHand = GetMesh()->GetBodyInstance("hand_r");
}


void AMyCharacter::AttackStart()
{	
	FTransform T = this->RHand->GetUnrealWorldTransform();
	this->SaveAttackStartPos = T.GetLocation();

	UE_LOG(LogTemp, Warning, TEXT("AttackStart !!!!!!!!!!!!!!!!!!!!!  %f %f %f"),
		T.GetLocation().X,
		T.GetLocation().Y,
		T.GetLocation().Z);
}

void AMyCharacter::AttackEnd()
{
	FTransform T = this->RHand->GetUnrealWorldTransform();

	/*
	TArray <struct FOverlapResult> OutOverlaps;
	GetWorld()->OverlapMultiByProfile(
		OutOverlaps,
		T.GetLocation(),
		T.GetRotation(),
		FName(TEXT("Pawn")), FCollisionShape::MakeSphere(50.0f));
	*/

	TArray <struct FHitResult> OutOverlaps;

	GetWorld()->SweepMultiByProfile(
		OutOverlaps, 
		this->SaveAttackStartPos,
		T.GetLocation(), 
		T.GetRotation(),
		FName(TEXT("Pawn")), 
		FCollisionShape::MakeSphere(50.0f));


	for (const FHitResult& e : OutOverlaps)
	{
		UE_LOG(LogTemp, Warning, TEXT("Collision !!!!!!!!!!!!!!!!!!!!!  %s"),
			*e.GetActor()->GetFullName(),
			*e.GetComponent()->GetFullName());
	}


	UE_LOG(LogTemp, Warning, TEXT("AttackEnd !!!!!!!!!!!!!!!!!!!!!  %f %f %f"),
		T.GetLocation().X,
		T.GetLocation().Y,
		T.GetLocation().Z);


	DrawDebugLine(GetWorld(), this->SaveAttackStartPos, T.GetLocation(), FColor::Emerald, true, -1, 0, 10);

}
```
### Add animation notify
애니메이션의 특정 상태에서 notify 이벤트를 발생시키도록 AttackStart, AttackEnd notify를 생성해준다.
AttackStart는 공격을 위해 주먹을 높이 든 상태이고,
AttackEnd는 공격 충돌 판정의 끝 상태이다.

![image-center](/assets/images/unreal-collision-by-animnotify-addnotify.png){: .align-center}


### Animation blueprint
애니메이션 몽타주 블루프린트에서 notify를 받을 시 c++ 코드에 정의된 함수를 호출해주도록 한다.
![image-center](/assets/images/unreal-collision-by-animnotify-animblueprint.png){: .align-center}


### result
주먹을 휘두르면 그 궤적에 포함된 대상 객체들을 쿼리해 온다. 
이후에 이 객체들을 활용하여 원하는 컨텐츠 코드를 작성할 수 있을 것이다.
![image-center](/assets/images/unreal-collision-by-animnotify-result.png){: .align-center}

