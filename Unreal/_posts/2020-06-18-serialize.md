---
title: "[Unreal] Serialize"
categories:
  - Serialize
---


#### File Read Write

```c++
void FNFaceModule::WriteTargetToFile()
{
	FString FullPath = FString::Printf(TEXT("%s%s"), *FPaths::ProjectSavedDir(), TEXT("NFaceConvertData.txt"));
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

void FNFaceModule::ReadTargetFromFile()
{
	FString FullPath = FString::Printf(TEXT("%s%s"), *FPaths::ProjectSavedDir(), TEXT("NFaceConvertData.txt"));
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


