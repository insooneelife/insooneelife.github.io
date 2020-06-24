---
title: "[Unreal] Serialize"
categories:
  - Serialize
---


#### File Read Write
파일 읽기 쓰기 예제이다.

#### Read/Write 기본 예제

```c++
void MyModule::WriteTargetToFile()
{
	FString FullPath = FString::Printf(TEXT("%s%s"), *FPaths::ProjectSavedDir(), TEXT("save.txt"));
	FArchive* ArWriter = IFileManager::Get().CreateFileWriter(*FullPath);


	FString T1 = FString(TEXT("AAA"));
	FString T2 = FString(TEXT("BBB"));
	if (ArWriter)
	{
		*ArWriter << T1;
		*ArWriter << T2;
		ArWriter->Close();
		delete ArWriter;
		ArWriter = NULL;
	}
}

void MyModule::ReadTargetFromFile()
{
	FString FullPath = FString::Printf(TEXT("%s%s"), *FPaths::ProjectSavedDir(), TEXT("save.txt"));
	TSharedPtr<FArchive> FileReader = MakeShareable(IFileManager::Get().CreateFileReader(*FullPath));
	if (FileReader.IsValid())
	{
		FString Temp1;
		FString Temp2;

		*FileReader.Get() << Temp1;
		*FileReader.Get() << Temp2;

		FileReader->Close();
		UE_LOG(LogTemp, Warning, TEXT("%s %s"),*Temp1,*Temp2);

	}
}
```


#### MyObject Serialize
Object의 read/write의 경우 일반적으로는 지원하지 않으므로, 경로를 따로 저장하는 방향으로 개발하는 것이 좋다.

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

UCLASS()
class MYPLUGIN_API UMyObject : public UObject
{
	GENERATED_BODY()
	
public:
	UMyObject() {}


	UPROPERTY(EditAnywhere, Category = "Voice Characteristics")
		EType Language;

	UPROPERTY(EditAnywhere, Category = "Conversion")
		FString SaveAssetTo;

	// 참조 형태의 변수는 Serialize 후 파일 Read/Write 과정에 문제가 생길 수 있다.
	// 따로 경로만 저장하는 형태로 개발하거나,
	// Config에 저장하는 형태로 개발하는것이 좋다.
	//UPROPERTY(EditAnywhere, Category = "Conversion")
	//	USkeleton* TargetSkeleton;

	UPROPERTY(EditAnywhere, Category = "Conversion")
		bool bImportAudio;

	UPROPERTY(EditAnywhere, Category = "Conversion")
		bool bInsertPlaySoundNotify;

	UPROPERTY(EditAnywhere, Category = "FilesToDump")
		bool bPhoneme;

	UPROPERTY(EditAnywhere, Category = "FilesToDump")
		bool bAnimClip;

	UPROPERTY(EditAnywhere, Category = "FilesToDump")
		bool bConversionLog;
};


```



#### Object Serialize and Read Write
UObject Serialize 후 파일에 Read/Write 하는 예제이다.
UPROPERTY로 정의된 변수들은 모두 기본적으로 직렬화된다.

```c++

// Read/Write 함수를 호출하는 부분
FReply SMyWindow::OnClickAddFiles()
{
	UE_LOG(LogTemp, Warning, TEXT("#### OnClickAddFiles ####"));
		
	FString FullPath = FString::Printf(TEXT("%s%s"), *FPaths::ProjectSavedDir(), TEXT("save.txt"));
	Write(FullPath, TargetObject);

	UMyObject* ReadData = NewObject<UMyObject>();
	Read(FullPath, ReadData);

	UE_LOG(LogTemp, Warning, TEXT("ReadData %d %s %d %d %d %d %d"),
		ReadData->Language,
		*ReadData->SaveAssetTo,
		ReadData->bAnimClip,
		ReadData->bImportAudio,
		ReadData->bInsertPlaySoundNotify,
		ReadData->bPhoneme,
		ReadData->bConversionLog
	);

	return FReply::Handled();
}


void SMyWindow::Write(const FString& FullPath, UObject* Data)
{
	TSharedPtr<FArchive> FileWriter = MakeShareable(IFileManager::Get().CreateFileWriter(*FullPath));

	if (FileWriter.IsValid())
	{
		TArray<uint8> SaveData;
		FMemoryWriter MemoryAr(SaveData, true);
		Data->Serialize(MemoryAr);

		*FileWriter << SaveData;
		FileWriter->Close();
	}
}


void SMyWindow::Read(const FString& FullPath, UObject* Data)
{
	TSharedPtr<FArchive> FileReader = MakeShareable(IFileManager::Get().CreateFileReader(*FullPath));

	if (FileReader.IsValid())
	{
		TArray<uint8> ReadData;
		*FileReader << ReadData;
		FMemoryReader MemoryAr(ReadData, true);

		Data->Serialize(MemoryAr);
		FileReader->Close();
	}
}
```


