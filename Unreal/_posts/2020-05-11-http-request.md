---
title: "[Unreal] HTTP Request"
categories:
  - HTTP
---


### Unreal http request 
언리얼에서 http request를 하는 예제이다.


#### header
```c++
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Runtime/Online/HTTP/Public/Http.h"
#include "MyHTTPActor.generated.h"

UCLASS()
class FACEFXEXAMPLE_API AMyHTTPActor : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AMyHTTPActor(const class FObjectInitializer& ObjectInitializer);

	FHttpModule* Http;

  // http 요청에 사용할 함수
	UFUNCTION()
	void MyHttpCall(); 

	void OnResponseReceived(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bWasSuccessful);

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

};

```

#### source
```c++
#include "MyHTTPActor.h"
#include "Dom/JsonObject.h"
#include "Serialization/JsonReader.h"
#include "Serialization/JsonSerializer.h"

// Sets default values
AMyHTTPActor::AMyHTTPActor(const class FObjectInitializer& ObjectInitializer)
	: Super(ObjectInitializer)
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
	Http = &FHttpModule::Get();
}

// Called when the game starts or when spawned
void AMyHTTPActor::BeginPlay()
{
	MyHttpCall();
	Super::BeginPlay();
}

// Called every frame
void AMyHTTPActor::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
}

void AMyHTTPActor::MyHttpCall()
{
	TSharedRef<IHttpRequest> Request = Http->CreateRequest();
	Request->OnProcessRequestComplete().BindUObject(this, &AMyHTTPActor::OnResponseReceived);

	//This is the url on which to process the request

	Request->SetURL("http://unreal.mywebcommunity.org/getInt.php");
	Request->SetVerb("GET");
	Request->SetHeader(TEXT("User-Agent"), "X-UnrealEngine-Agent");
	Request->SetHeader("Content-Type", TEXT("application/json"));
	Request->ProcessRequest();
}


void AMyHTTPActor::OnResponseReceived(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bWasSuccessful)
{
	//Create a pointer to hold the json serialized data
	TSharedPtr<FJsonObject> JsonObject;

	//Create a reader pointer to read the json data
	TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(Response->GetContentAsString());

	//Deserialize the json data given Reader and the actual object to deserialize
	if (FJsonSerializer::Deserialize(Reader, JsonObject))
	{
		//Get the value of the json object by field name
		int32 recievedInt = JsonObject->GetIntegerField("customInt");

		UE_LOG(LogTemp, Warning, TEXT("HTTP request result   customInt : %d"), recievedInt);
	}
}
```





