---
title: "[Unreal] Serialize"
categories:
  - Serialize
---


#### File Read Write
파일 읽기 쓰기 예제이다.

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

#### Object Serialize and Read Write
UObject Serialize 예제이다.
UPROPERTY로 정의된 변수들은 모두 기본적으로 직렬화된다.

```c++
void MyModule::Write()
{
	FString FullPath = FString::Printf(TEXT("%s%s"), *FPaths::ProjectSavedDir(), TEXT("save.txt"));
	TSharedPtr<FArchive> FileWriter = MakeShareable(IFileManager::Get().CreateFileWriter(*FullPath));

	if (FileWriter.IsValid())
	{
		UMyData* Data = NewObject<UMyData>();
		Data->StringData = FString(TEXT("Game/Temp/BlaBla/"));
		Data->bData = true;

		TArray<uint8> SaveData;
		FMemoryWriter MemoryWriter(SaveData, true);
		Data->Serialize(MemoryWriter);

		*FileWriter << SaveData;
		FileWriter->Close();
	}
}

void MyModule::Read()
{
	FString FullPath = FString::Printf(TEXT("%s%s"), *FPaths::ProjectSavedDir(), TEXT("save.txt"));
	TSharedPtr<FArchive> FileReader = MakeShareable(IFileManager::Get().CreateFileReader(*FullPath));

	if (FileReader.IsValid())
	{
		UMyData* Data = NewObject<UMyData>();
		TArray<uint8> ReadData;
		*FileReader << ReadData;
		FMemoryReader MemoryReader(ReadData);

		Data->Serialize(MemoryReader);

		UE_LOG(LogTemp, Warning, TEXT("%s %d"), *Data->StringData, Data->bData);

		FileReader->Close();
	}
}
```


