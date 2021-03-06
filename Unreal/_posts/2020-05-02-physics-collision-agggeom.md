---
title: "[Unreal] Physics Collision AggregateGeom"
categories:
  - Physics
  - Collision
---

### Physics Collision with AggregateGeom Shapes
캐릭터가 공격할때 오른손 캡슐에 의한 충돌판정을 위한 예제이다.

#### physics asset
먼저 충돌판정에 사용할 BodyInstance이다. 
이후 코드를 통해 이 BodyInstance의 궤적에 따른 충돌을 판정할 것이다.
![image-center](/assets/images/unreal-physics-collision-agggeom-physics-asset.png){: .align-center}

#### physics asset



#### header

```c++
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

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;


private:
	UCapsuleComponent * Capsule;

	FBodyInstance* RHand;

	FKSphylElem SaveBodySetup;
};
```
#### source

```c++
// Fill out your copyright notice in the Description page of Project Settings.


#include "MyCharacter.h"
#include "Components/SkeletalMeshComponent.h"
#include "Components/StaticMeshComponent.h"
#include "PhysicsEngine/BodySetup.h"
#include "Components/CapsuleComponent.h"

#include "DrawDebugHelpers.h"

// Sets default values
AMyCharacter::AMyCharacter()
{
 	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
}

// Called when the game starts or when spawned
void AMyCharacter::BeginPlay()
{
	Super::BeginPlay();
	
	this->Capsule = Cast<UCapsuleComponent>(this->RootComponent->GetChildComponent(3));
	this->Capsule->ResetRelativeTransform();

	if (this->Capsule != nullptr)
	{
		UE_LOG(LogTemp, Warning, TEXT("%s"), *this->Capsule->GetName());
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("nullptr sphere mesh"));
	}


	this->RHand = GetMesh()->GetBodyInstance("hand_r");
	
	if (this->RHand != nullptr)
	{
		const FKAggregateGeom& Geom = this->RHand->BodySetup->AggGeom;
		const TArray<FKSphylElem>& Elems = Geom.SphylElems;

		for (const FKSphylElem& e : Elems)
		{
			this->SaveBodySetup = e;

			FVector Center = e.Center;
			FRotator Rotate = e.Rotation;
			float Radius = e.Radius;
			float Length = e.Length;

			this->Capsule->SetCapsuleSize(Radius, Length);
		}
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("hand is null"));
	}
	

}

// Called every frame
void AMyCharacter::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (this->RHand != nullptr)
	{
		FTransform t = RHand->GetUnrealWorldTransform();

		UE_LOG(LogTemp, Warning, TEXT("%s"), *this->Capsule->GetName());
		
		t.SetRotation(t.GetRotation() * FQuat(SaveBodySetup.Rotation));
		this->Capsule->SetWorldTransform(t);
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("hand is null"));
	}
}

// Called to bind functionality to input
void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

}

```


#### blueprint
블루프린트 예제이다.
1. OnComponentBeginOverlap 이벤트가 발생했을 때
2. 현재 IsAttacking 상태일 때
3. 충돌 대상이 자기자신이 아닐 때 (자기자신의 다른 컴포넌트와 충돌 이벤트가 발생하는 경우가 있음)
4. 타겟 컴포넌트의 tag가 Root일 때
화면에 대상에 대한 정보를 출력해준다.

![image-center](/assets/images/unreal-physics-collision-agggeom-blueprint.png){: .align-center}


#### result
결과 화면이다.
![image-center](/assets/images/unreal-physics-collision-agggeom-physics-result.png){: .align-center}
