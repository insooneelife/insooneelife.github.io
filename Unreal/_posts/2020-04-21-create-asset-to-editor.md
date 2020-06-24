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

#include "UnrealEd.h"
#include "UObject/Package.h"
#include "AssetRegistryModule.h"
#include "MyPrimaryDataAsset.h"

...

void ATestProjectGameMode::StartPlay()
{
	Super::StartPlay();

	FString AssetName = TEXT("MyPrimaryDataAsset");
	FString PackageName = TEXT("/Game/");
	PackageName += AssetName;

	UPackage* Package = CreatePackage(NULL, *PackageName);
	Package->FullyLoad();
	
	UMyPrimaryDataAsset* NewAsset = NewObject<UMyPrimaryDataAsset>(
		Package, *AssetName, RF_Public | RF_Standalone | RF_MarkAsRootSet);
	
	NewAsset->ItemName = TEXT("NewAssetName");

	Package->MarkPackageDirty();
	FAssetRegistryModule::AssetCreated(NewAsset);

	FString PackageFileName = FPackageName::LongPackageNameToFilename(
		PackageName, FPackageName::GetAssetPackageExtension());

	bool bSaved = UPackage::SavePackage(
		Package,
		NewAsset,
		EObjectFlags::RF_Public | EObjectFlags::RF_Standalone,
		*PackageFileName, 
		GError, nullptr, true, true, SAVE_NoError);

	TArray<UObject*> ObjectsToSync;
	ObjectsToSync.Add(NewAsset);
	GEditor->SyncBrowserToObjects(ObjectsToSync);

}
```

#### 실행 결과
다음과 같이 에셋이 에디터에 저장된다.
![image-center](/assets/images/unreal-create-asset-to-editor-result.png){: .align-center}




#### Asset 로드
모듈에서 Asset을 로드해야하는 경우가 생길 수 있다.
(로드 시 주의할 점은 Asset은 StartupModule 때 준비되지 않았을 수도 있다.)
```c++
	FAssetRegistryModule& AssetRegistryModule =
		FModuleManager::LoadModuleChecked<FAssetRegistryModule>("AssetRegistry");
	FAssetData AssetData =
		AssetRegistryModule.Get().GetAssetByObjectPath(TEXT("/Game/MyPrimaryDataAsset.MyPrimaryDataAsset"));

	if (AssetData.IsValid())
	{
		UMyPrimaryDataAsset* Data = Cast<UMyPrimaryDataAsset> (AssetData.GetAsset());
		UE_LOG(LogTemp, Warning, TEXT("#######  Success %s #######"), *Data->ItemName);	
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("#######  Failed  #######"));
	}
```
