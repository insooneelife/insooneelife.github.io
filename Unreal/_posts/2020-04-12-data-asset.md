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
이제 editor에서 실제 데이터 테이블 에셋 파일을 생성한다.
콘텐츠 브로우저 -> 신규추가 -> 기타 -> 데이터 테이블
구조체 선택 창이 뜨면 이전에 제작한 class를 선택한다.

생성된 table에 .csv파일을 로드한다.

##csv example
```
Name,		x,		y,		z,		quatX,		quatY,		quatZ,		quatW
a,		1,		-1, 		0.5, 		0,		-1,		-1.2, 		2.2
b,		2,		-3,		1.5,		0,		-1,		-1.6,		3.2
```

##c++ code에서 data asset 사용하기

```c++
#include "Public/CustomDataAsset.h"
#include "UObject/ConstructorHelpers.h"
...
	UDataTable* _dataTable;
...


```


```c++
static ConstructorHelpers::FObjectFinder<UDataTable> DataTable(TEXT("/Game/Data/PhysicsPoseDataTable.PhysicsPoseDataTable"));
if (DataTable.Succeeded())
{
	_dataTable = DataTable.Object;

	TArray<FPhysicsPoseData*> arr;
	_dataTable->GetAllRows<FPhysicsPoseData>(TEXT("GetAllRows"), arr);

	for (int i = 0; i < arr.Num(); ++i)
	{
	UE_LOG(LogTemp, Warning, TEXT("row loc : %f %f %f  quat : %f %f %f %f"), 
		arr[i]->x, arr[i]->y, arr[i]->z,
		arr[i]->quatX, arr[i]->quatY, arr[i]->quatZ, arr[i]->quatW);
	}
}
```


