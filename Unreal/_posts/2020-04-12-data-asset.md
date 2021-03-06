---
title: "[Unreal] Data"
categories:
  - Data
---

### Data Asset

#### data asset
Unreal engine에서 커스텀 데이터를 이용하고 싶은 경우 data asset을 이용하면 효과적이다.

Data asset을 이용하기 위해서 먼저 사용하고자 하는 data 형태를 c++로 작성한다. 

```c++
#include "Engine/DataAsset.h"

UCLASS()
class TESTWORLD_API UCustomDataAsset : public UPrimaryDataAsset
{
	GENERATED_BODY()
public:	
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
```
이제 editor에서 실제 데이터 테이블 에셋 파일을 생성한다.
콘텐츠 브로우저 -> 신규추가 -> 기타 -> 데이터 에셋
클래스 선택 창이 뜨면 CustomDataAsset를 선택한다.
만들어진 data asset에 원하는 데이터를 채운다.


#### c++ code에서 data asset 사용하기
```c++
#include "Public/CustomDataAsset.h"
#include "UObject/ConstructorHelpers.h"

static ConstructorHelpers::FObjectFinder<UDataAsset> DataAsset(TEXT("/Game/Data/CustomDataAsset.CustomDataAsset"));
if (DataAsset.Succeeded())
{
	UDataAsset* dataAsset = DataAsset.Object;
	UCustomDataAsset* data = Cast<UCustomDataAsset>(dataAsset);

	UE_LOG(LogTemp, Warning, TEXT(" Data asset loc : %f %f %f"), 
		data->loc.X, data->loc.Y, data->loc.Z);
}
```



#### data table
대량의 데이터를 table 형태로 관리해야하는 상황도 생길 수 있다.
이런 경우 unreal engine의 data table을 사용하면 유용하다.

```c++
#include "Engine/DataTable.h"

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

#### csv example
```
Name,	x,	y,	z,	quatX,	quatY,	quatZ,	quatW
a,	1,	-1, 	0.5, 	0,	-1,	-1.2, 	2.2
b,	2,	-3,	1.5,	0,	-1,	-1.6,	3.2
```

#### c++ code에서 data table 사용하기


```c++
#include "Public/CustomDataAsset.h"
#include "UObject/ConstructorHelpers.h"

// load data table
static ConstructorHelpers::FObjectFinder<UDataTable> DataTable(
TEXT("/Game/Data/PhysicsPoseDataTable.PhysicsPoseDataTable"));
if (DataTable.Succeeded())
{
	UDataTable* dataTable = DataTable.Object;

	TArray<FPhysicsPoseData*> arr;
	dataTable->GetAllRows<FPhysicsPoseData>(TEXT("GetAllRows"), arr);

	// read data
	for (int i = 0; i < arr.Num(); ++i)
	{
		UE_LOG(LogTemp, Warning, TEXT("row loc : %f %f %f  quat : %f %f %f %f"), 
			arr[i]->x, arr[i]->y, arr[i]->z,
			arr[i]->quatX, 
			arr[i]->quatY, 
			arr[i]->quatZ, 
			arr[i]->quatW);
	}

	// write data
	for (int i = 0; i < arr.Num(); ++i)
	{
		arr[i]->x = 1;
		arr[i]->y = 0;
		arr[i]->z = 3;

		UE_LOG(LogTemp, Warning, TEXT("row loc : %f %f %f  quat : %f %f %f %f"),
			arr[i]->x, arr[i]->y, arr[i]->z,
			arr[i]->quatX,
			arr[i]->quatY,
			arr[i]->quatZ,
			arr[i]->quatW);
	}
}
```

#### add item to table

```c++
void UtilsData::WritePoseDataTable()
{
	static ConstructorHelpers::FObjectFinder<UDataTable> DataTable(
		TEXT("/Game/Data/PhysicsPoseDataTable.PhysicsPoseDataTable"));
	if (DataTable.Succeeded())
	{
		UDataTable* dataTable = DataTable.Object;

		int idx = 0;
		for (int i = 0; i < AndroidAnimPhysicsData.Num(); ++i)
		{
			const PhysicsPoseData& poseData = AndroidAnimPhysicsData[i];
			float time = poseData.seqTime;

			for (int j = 0; j < poseData.data.Num(); ++j)
			{
				const PhysicsData& data = poseData.data[j];
				FName name(*FString::FromInt(idx));
				FPhysicsPoseDataTableRow row;

				row.time = time;
				row.bname = data.bname;
				row.boneIdx = data.boneIdx;
				row.parentBoneIdx = data.parentBoneIdx;
				row.x = data.loc.X;
				row.y = data.loc.Y;
				row.z = data.loc.Z;
				row.quatX = data.rot.X;
				row.quatY = data.rot.Y;
				row.quatZ = data.rot.Z;
				row.quatW = data.rot.W;
				row.jointAngX = data.jointAng.X;
				row.jointAngY = data.jointAng.Y;
				row.jointAngZ = data.jointAng.Z;
				row.velX = data.vel.X;
				row.velY = data.vel.Y;
				row.velZ = data.vel.Z;
				row.jointAngVelX = data.angVel.X;
				row.jointAngVelY = data.angVel.Y;
				row.jointAngVelZ = data.angVel.Z;
				row.physicsParentName = data.physicsParentName;
				row.physicsParentIdx = data.physicsParentIdx;

				dataTable->AddRow(name, row);
				idx++;
			}
		}
	}
}

```

