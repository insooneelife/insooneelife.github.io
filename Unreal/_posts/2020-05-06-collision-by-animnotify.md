---
title: "[Unreal] Collision By Animation Notify"
categories:
  - Animation
  - Collision
---

header
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


source
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
