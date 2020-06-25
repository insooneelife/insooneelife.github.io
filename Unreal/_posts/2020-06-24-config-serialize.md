---
title: "[Unreal] Config Serialize"
categories:
  - Serialize
  - Config
---


#### Config Serialize
Config 파일을 이용하여 UObject의 데이터를 파일에 저장할 수 있다.
그리고 이 방법으로는 일반적인 방법으로는 read/write가 불가능한 asset과 같은 형태의 데이터도 저장이 가능하다.


#### 데이터 정의
```c++

#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "Animation/Skeleton.h"
#include "MyObject.generated.h"


UENUM(BlueprintType)
enum class EType : uint8
{
	Type1,
	Type2
};

// Editor.ini에 저장한다.
UCLASS(Config = Editor)
class MYPLUGIN_API UMyObject : public UObject
{
	GENERATED_BODY()
	
public:
	UMyObject() {}


	UPROPERTY(EditAnywhere, Category = "Voice Characteristics", Config)
		EType Language;

	UPROPERTY(EditAnywhere, Category = "Conversion", Config)
		FString SaveAssetTo;

	UPROPERTY(EditAnywhere, Category = "Conversion", Config)
		TSoftObjectPtr<USkeleton> TargetSkeleton;

	UPROPERTY(EditAnywhere, Category = "Conversion", Config)
		bool bImportAudio;

	UPROPERTY(EditAnywhere, Category = "Conversion", Config)
		bool bInsertPlaySoundNotify;

	UPROPERTY(EditAnywhere, Category = "FilesToDump", Config)
		bool bPhoneme;

	UPROPERTY(EditAnywhere, Category = "FilesToDump", Config)
		bool bAnimClip;

	UPROPERTY(EditAnywhere, Category = "FilesToDump", Config)
		bool bConversionLog;
};
```

#### Config File Save
다음과 같이 Config 에 저장할 수 있다.
```c++
...
TargetObject->SaveConfig();
...
```

#### 결과 화면
![image-center](/assets/images/unreal-config-serialize-result.png){: .align-left}



