# UE5_LightDetection

## Determines whether the character is under direct sunlight or in shadow.

### Developed with Unreal Engine 5

- [Overview](#overview)
- [Blueprint Version](#blueprint-version)
- [C++ Version](#c-version)


## Overview

![Light detection](https://github.com/paveldrobny/UE5_LightDetection/blob/master/InLight.png)
![Shadow detection](https://github.com/paveldrobny/UE5_LightDetection/blob/master/InShadow.png)

## Blueprint Version

![BPCode](https://github.com/paveldrobny/UE5_LightDetection/blob/master/BP_Demo.png)

## C++ Version

### CustomCharacter.h

```cpp

class ADirectionalLight;

class YOUR_API ACustomCharacter : public ACharacter
{
    GENERATED_BODY()

public:
    // Sets default values for this character's properties
	ACustomCharacter();

private:
	 UPROPERTY()
	 ADirectionalLight* SunLightRef;

	 void UpdateSunDetection();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;
}

```

### CustomCharacter.cpp

```cpp
#include "Kismet/GameplayStatics.h"
#include "Engine/DirectionalLight.h"
#include "DrawDebugHelpers.h"
#include "Engine/Engine.h" 

void ACustomCharacter::BeginPlay()
{
	Super::BeginPlay();

	if (!SunLightRef)
	{
		SunLightRef = Cast<ADirectionalLight>(
			UGameplayStatics::GetActorOfClass(GetWorld(), ADirectionalLight::StaticClass())
		);
	}
}

void ACustomCharacter::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	UpdateSunDetection();
}

void ACustomCharacter::UpdateSunDetection()
{
	if (!SunLightRef)
	{
		return;
	}

	FVector SunDir = -SunLightRef->GetActorForwardVector();
	FVector Start = GetActorLocation();
	FVector End = Start + (SunDir * 10000);

	FHitResult Hit;

	bool bHit = GetWorld()->LineTraceSingleByChannel(Hit, Start, End, ECC_Visibility);

	bool bPersistent = false;
	float LifeTime = 0.1f;
	uint8 DepthPriority = 0;
	float Thickness = 1.0f;

	DrawDebugLine(
		GetWorld(),
		Start,
		Hit.TraceEnd,
		FColor::Red,
		bPersistent,
		LifeTime,
		DepthPriority,
		Thickness
	);

	AActor* HitActor = Hit.GetActor();

	if (IsValid(HitActor))
	{
		GEngine->AddOnScreenDebugMessage(-1, 0.5f, FColor::Green, TEXT("InShadow"));
	}
	else
	{
		GEngine->AddOnScreenDebugMessage(-1, 0.5f, FColor::Red, TEXT("InLight"));
	}
}
```
