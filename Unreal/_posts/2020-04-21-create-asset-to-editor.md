---
title: "[Unreal] Create Asset To Editor"
categories:
  - Asset
---

### Create Asset To Editor
Unreal 에디터에 c++을 이용하여 원하는 에셋을 생성하는 예제이다.


#### target asset
먼저 에디터에 생성되길 원하는 에셋을 정의한다.

Data asset을 이용하기 위해서 먼저 사용하고자 하는 data 형태를 c++로 작성한다. 

```c++
#include "CoreMinimal.h"
#include "Engine/DataAsset.h"
#include "MyPrimaryDataAsset.generated.h"

UCLASS()
class TESTPROJECT_API UMyPrimaryDataAsset : public UPrimaryDataAsset
{
	GENERATED_BODY()
	
public:
	UPROPERTY(EditAnywhere)
		FString ItemName;

	UPROPERTY(EditAnywhere)
		UBlueprint* ItemBlueprint;

	UPROPERTY(EditAnywhere)
		FColor ItemColor;

	UPROPERTY(EditAnywhere)
		FVector Loc;

	UPROPERTY(EditAnywhere)
		FQuat Rot;
};
```

#### asset factory
이제 에셋을 생성시켜줄 factory class를 정의한다. 
UFactory class를 상속받은 새 클래스를 만든다.

```c++
#include "CoreMinimal.h"
#include "Factories/Factory.h"
#include "MyPrimaryDataAsset.h"
#include "MyFactory.generated.h"

UCLASS()
class TESTPROJECT_API UMyFactory : public UFactory
{
	GENERATED_BODY()
public:
	UMyFactory(const FObjectInitializer& ObjectInitializer);
	virtual UObject* FactoryCreateNew(UClass* InClass, UObject* InParent, FName InName, EObjectFlags Flags, UObject* Context, FFeedbackContext* Warn) override;


	// Helps us to know if we are create a new Object or just saving an existing one.
	UMyPrimaryDataAsset* CreatedObjectAsset;
};
```

```c++
#include "MyFactory.h"
#include "UnrealEd.h"

UMyFactory::UMyFactory(const FObjectInitializer& ObjectInitializer) :Super(ObjectInitializer)
{
	bCreateNew = true;
	bEditAfterNew = true;
	SupportedClass = UMyPrimaryDataAsset::StaticClass();
}

UObject* UMyFactory::FactoryCreateNew(UClass* Class, UObject* InParent, FName Name, EObjectFlags Flags, UObject* Context, FFeedbackContext* Warn)
{

	if (CreatedObjectAsset != nullptr)
	{
		CreatedObjectAsset->SetFlags(Flags | RF_Transactional);
		CreatedObjectAsset->Modify();
		CreatedObjectAsset->Rename(*Name.ToString(), InParent);
	}
	else
	{
		CreatedObjectAsset = NewObject<UMyPrimaryDataAsset>(InParent, Class, Name, Flags | RF_Transactional);
	}
	return CreatedObjectAsset;
}
```


#### gamemode에서 에셋 생성
```c++
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "TestProjectGameMode.generated.h"

UCLASS(minimalapi)
class ATestProjectGameMode : public AGameModeBase
{
	GENERATED_BODY()

public:
	ATestProjectGameMode();

	virtual void StartPlay() override;
};
```



```c++
#include "TestProjectGameMode.h"
#include "UObject/ConstructorHelpers.h"

#include "Engine/Engine.h"
#include "Public/MyFactory.h"
#include "IAssetTools.h"
#include "AssetToolsModule.h"
#include "UnrealEd.h"

void ATestProjectGameMode::StartPlay()
{
	Super::StartPlay();
  
	UMyFactory* NewFactory = NewObject<UMyFactory>();
	FAssetToolsModule& AssetToolsModule = FAssetToolsModule::GetModule();
	UObject* NewAsset = AssetToolsModule.Get().CreateAsset(NewFactory->GetSupportedClass(), NewFactory);
	TArray<UObject*> ObjectsToSync;
	ObjectsToSync.Add(NewAsset);
	GEditor->SyncBrowserToObjects(ObjectsToSync);
}
```


