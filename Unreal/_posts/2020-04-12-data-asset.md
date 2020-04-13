---
title: "[Unreal] Data"
categories:
  - Markup
  - Data
---

### Data Asset

#### data asset
Unreal engine에서 커스텀 데이터를 이용하고 싶은 경우 data asset을 이용하면 효과적이다.
Data asset을 이용하기 위해서 먼저 사용하고자 하는 data 형태를 c++로 작성한다. 

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

#### data table
대량의 데이터를 table 형태로 관리해야하는 상황도 생길 수 있다.
이런 경우 unreal engine의 data table을 사용하면 유용하다.

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
