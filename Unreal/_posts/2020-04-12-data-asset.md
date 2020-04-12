---
title: "[Unreal] Data"
categories:
  - Markup
  - Data
---

### Data Asset
#### data table

```c++
#include "CoreMinimal.h"
#include "Engine/DataAsset.h"
#include "Engine/DataTable.h"
#include "MyPrimaryDataAsset.generated.h"

USTRUCT()
struct FPhysicsPoseData : public FTableRowBase
{
	GENERATED_USTRUCT_BODY()

	UPROPERTY(EditAnywhere)
	float x;

	UPROPERTY(EditAnywhere)
	float y;

	UPROPERTY(EditAnywhere)
	float z;

	UPROPERTY(EditAnywhere)
	float quatX;

	UPROPERTY(EditAnywhere)
	float quatY;

	UPROPERTY(EditAnywhere)
	float quatZ;

	UPROPERTY(EditAnywhere)
	float quatW;

};
```

#### data asset

```c++
USTRUCT()
struct FItemInfo {

	GENERATED_USTRUCT_BODY()

	UPROPERTY(EditAnywhere)
	FString itemName;

	UPROPERTY(EditAnywhere)
	UBlueprint* itemBlueprint;

	UPROPERTY(EditAnywhere)
	FColor itemColor;

	UPROPERTY(EditAnywhere)
	FVector loc;

	UPROPERTY(EditAnywhere)
	FQuat rot;
};

/**
 * 
 */
UCLASS()
class TESTPHYSICS_API UMyPrimaryDataAsset : public UPrimaryDataAsset
{
	GENERATED_BODY()

	UPROPERTY(EditAnywhere)
	FItemInfo data;
};
```
