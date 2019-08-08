# 1.日志

### 1.1 自定义日志类型

```
.h 声明 DECLARE_LOG_CATEGORY_EXTERN(FuxxModule, Log, All);
.cpp 定义 DEFINE_LOG_CATEGORY(FuxxModule);
UE_LOG(FuxxModule, Warning, TEXT("自定义日志类型")); //即可使用
```

### 1.2 日志基本输出

```
UE_LOG(LogTemp, Log, TEXT("output Log"));//Log,Display,Warning,Error 四种类型

UE_LOG(LogTemp, Warning, TEXT("int output %d"), 123);
UE_LOG(LogTemp, Warning, TEXT("float output %f"), 3.14f);
UE_LOG(LogTemp, Warning, TEXT("String output %s"), TEXT("hello"));
FVector vec(1, 2, 3);
UE_LOG(LogTemp, Warning, TEXT("vector output %s"), *vec.ToString());
```

### 1.3

```
//向控制台输出，~~查看控制台输出
GetWorld()->GetFirstPlayerController()->ClientMessage(TEXT("hello"));
//屏幕打印
GEngine->AddOnScreenDebugMessage(-1, 10.0f, FColor::Red, TEXT("output to screen"));
```

+++



# 2.UClass

```
#include "xxx.generated.h" //这个头文件是必须的并且是最后一行#include，否则报错。用于蓝图反射。

UCLASS(Blueprintable)//蓝图可用
class MYSTUDY_API UUserProfile : public UObject
{
	GENERATED_BODY()
        
public:
	//护甲值 该注释可在编辑器属性上有提示
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "UserData")
		float Armor;
}    
```

+++

```
USTRUCT(BlueprintType)
struct FPlayerInfo
{
	GENERATED_USTRUCT_BODY()
	/* 玩家编号 */
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
		int32 PlayerNumber;
	/* 玩家颜色 */
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
        FColor PlayerColor;
};
    
UENUM(BlueprintType)
enum EStatus
{
	SAYHELLO		UMETA(DisplayName = "Say hello"),
	//UMETA(DisplayName = "Say hello") 展示名重命名  Say hello
	CREATEUOBJECT   UMETA(DisplayName = "Create a UserProfile"),
};
    
UCLASS()//以继承AActor，AActor继承UObject
class MYSTUDY_API ACoolObject : public AActor
{
	GENERATED_BODY()		
		
public:	
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "Your Category")
	float DefaultsOnly;

	UPROPERTY(EditInstanceOnly, BlueprintReadWrite, Category = Your Category)
	float InstanceOnly;

	UFUNCTION(BlueprintCallable, Category = "ObjectFun")
	void YourFunc();

	UPROPERTY()//指针不会在运行时被回收
	class UUserProfile* UserProfile;

	//在cpp里获得一个蓝图的引用
	UPROPERTY(EditAnywhere, Category = "Profile")
	TSubclassOf<class UUserProfile> UserProfileClass;
	//cppNewObject用于创建UObject对象
	//NewObject<UUserProfile>(GetTransientPackage(), UserProfileClass);

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Profile")
	FPlayerInfo PlayerInfo;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Profile")
	TEnumAsByte<EStatus> CurrentStatus; //更小更安全，优化enum
	
};    
```

+++

	//ConditionalBeginDestroy() 会调用 BeginDestroy，主动销毁
	UserProfile->ConditionalBeginDestroy();
	
	//强制进行垃圾回收
	GEngine->ForceGarbageCollection(true);
+++

# 3.Memory And Smart Pointer

### 3.1

```
//自定义一个c++ Class
//假设我们保存游戏的玩家数
class FGameData{...}

UClass()...
//自动将对象变为持久，否则会定期清掉
UPROPERTY()
	class UAction* MyAction;

//声明共享指针
TSharedPtr<FGameData> GameDataPtr;
//声明弱指针
TWeakPtr<FGameData> GameDataWeakPtr;
```

+++

### 3.2 共享指针

```
//使用MakeShareable将对象付给共享指针
GameDataPtr = MakeShareable(new FGameData);

//获得共享指针指向对象的引用计数,GameDataPtr.GetSharedReferenceCount()
UE_LOG(LogTemp, Warning, TEXT("Ref count = %d"), GameDataPtr.GetSharedReferenceCount());

//共享指针 调用指向对象的成员函数
GameDataPtr->GetNumber();
	
//解引用 &dataRef == temp
FGameData &dataRef = *GameDataPtr;
FGameData* temp = GameDataPtr.Get();

GameDataPtr.Reset();//清空
```

+++

### 3.3 弱指针

```
//弱指针需要先用共享指针
GameDataPtr = MakeShareable(new FGameData);
GameDataWeakPtr = GameDataPtr;

//弱指针 调用指向对象的成员函数
GameDataWeakPtr.Pin()->GetNumber()

GameDataWeakPtr.IsValid()//判断指针是否有效
```

+++

# 4. Actor And Components



### 4.1 AFirstActor

```
//AFirstActor.h
UCLASS()
class MYSTUDY_API AFirstActor : public AActor
{
	//在UCLASS()声明的类内必须定义，目的是通过UHT插入额外代码来支持蓝图可见
	GENERATED_BODY() //在关卡里调用
	//GENERATED_UCLASS_BODY() //cpp里实现带参数的构造，在引擎初始化调用
	
public:	
	AFirstActor();	
protected:
	virtual void BeginPlay() override;
	virtual void BeginDestroy() override;
	virtual void Destroyed() override;

public:	
	virtual void Tick(float DeltaTime) override;

	UPROPERTY()//指针不被回收
	UStaticMeshComponent* Mesh;
};

```

+++

```
//AFirstActor.cpp
AFirstActor::AFirstActor()
{
	PrimaryActorTick.bCanEverTick = true;

	Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BaseMeshComponent"));

	RootComponent = Mesh;
	//效果同上，为根组件添加组件
	//Mesh->SetupAttachment(GetRootComponent());
	//Mesh->SetupAttachment(RootComponent);

	//构造方法可用
	ConstructorHelpers::FObjectFinder<UStaticMesh> MeshAsset = 		   ConstructorHelpers::FObjectFinder<UStaticMesh>(TEXT("StaticMesh'/Engine/EngineMeshes/Cube.Cube'"));	
	//auto MeshAsset = ConstructorHelpers::FObjectFinder<UStaticMesh>(TEXT(""));
	//ConstructorHelpers::FObjectFinder<UStaticMesh> MeshAsset(TEXT(""));

	if(MeshAsset.Object != nullptr)//if (MeshAsset.Succeeded())//效果相同	
		Mesh->SetStaticMesh(MeshAsset.Object);
}
```

---

### 4.2 SpawnActorGameMode 生成AFirstActor

```
//SpawnActorGameMode.h
UCLASS()
class MYSTUDY_API ASpawnActorGameMode : public AGameModeBase
{
	GENERATED_BODY()

public:
	virtual void BeginPlay() override;
	
	UPROPERTY()
	class AFirstActor* MyActor;

	UFUNCTION(BlueprintCallable, Category = ActorSpawn)
	void SpawnMyActor(FTransform SpawnTransform);
    
	UFUNCTION(BlueprintCallable, Category = ActorSpawn)
	void DestroyMyActor();

	UFUNCTION(BlueprintCallable, Category = "SpawnByTimer")
	void DestroyActorUseTimer(float LiveTime = 1.0f, bool IsLooping = false);

	FTimerHandle timeHandler;	
};
```

---

```
//SpawnActorGameMode.cpp
void ASpawnActorGameMode::SpawnMyActor(FTransform SpawnTransform)
{
	MyActor = GetWorld()->SpawnActor<AFirstActor>(AFirstActor::StaticClass(), SpawnTransform);	
	//MyActor->SetLifeSpan(3.0f);
}

void ASpawnActorGameMode::DestroyMyActor()
{
	if (MyActor != nullptr)
	{
		MyActor->Destroy();
		MyActor = nullptr;
	}
}

void ASpawnActorGameMode::DestroyActorUseTimer(float LiveTime /*= 1.0f*/, bool IsLooping /*= false*/)
{
	FTimerManager &timerManager = GetWorldTimerManager();
	//定时器
	timerManager.SetTimer(timeHandler, this, &ASpawnActorGameMode::DestroyMyActor, LiveTime, IsLooping);
}
```

+++

### 4.3 SpawnerComponent 创建一个场景组件，可在蓝图中添加

```
//SpawnerComponent.h
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class MYSTUDY_API USpawnerComponent : public USceneComponent
{
...
public:	
	// Called every frame 组件的Tick方法
	virtual void TickComponent(float DeltaTime, ELevelTick TickType,   FActorComponentTickFunction* ThisTickFunction) override;

	UPROPERTY(EditAnywhere, Category = SpawnClass)
		TSubclassOf<AStaticMeshActor> MeshActorClass;

	UFUNCTION(BlueprintCallable, Category = Spawner)
		void SpawnStaticMeshActor();	
};
```

---

```
//SpawnerComponent.cpp
void USpawnerComponent::SpawnStaticMeshActor()
{
	UWorld* world = GetWorld();
	if (world) {
		if (MeshActorClass)
		{
			FTransform componentTransform(this->GetComponentTransform());
			world->SpawnActor(MeshActorClass, &componentTransform);//SpawnActor
		}
	}
}
```

+++

### 4.4 OrbitMovementComponent 圆周运动，场景组件，在蓝图中添加

```
//OrbitMovementComponent.h
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class MYSTUDY_API UOrbitMovementComponent : public USceneComponent
{
...
public:	
	virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;

	UPROPERTY(EditAnywhere, Category = OrbitProperties)
		float Radius;

	float CurrentAngleValue;	//当前角度

	UPROPERTY(EditAnywhere, Category = OrbitProperties)
		float RotationSpeed;	//旋转速度
};
```

+++

```
//OrbitMovementComponent.cpp
UOrbitMovementComponent::UOrbitMovementComponent()
	:Radius(100),	//c++语法，可以在这设置默认参数
	CurrentAngleValue(0),
	RotationSpeed(50)
{
	...
}

// Called every frame
void UOrbitMovementComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	float CurrentRadiansValue = FMath::DegreesToRadians(CurrentAngleValue);//角度转弧度

	SetRelativeLocation(FVector(Radius * FMath::Cos(CurrentRadiansValue),
								Radius * FMath::Sin(CurrentRadiansValue),
								0)
	);
	//角度取余
	CurrentAngleValue = FMath::Fmod(CurrentAngleValue + DeltaTime * RotationSpeed, 360);
}
```

+++

### 4.5 RandomMovementComponent 球状区域跳跃

```
//RandomMovementComponent.h
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class MYSTUDY_API URandomMovementComponent : public UActorComponent
{
	GENERATED_BODY()

public:	
	// Sets default values for this component's properties
	URandomMovementComponent();

protected:
	// Called when the game starts
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;

	UPROPERTY(EditAnywhere, Category = Radius)
	float MovementRadius = 50.0f;
	
};
```

---

```
//RandomMovementComponent.cpp
// Called every frame
void URandomMovementComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	AActor* ownerActor = GetOwner();//得到该组件上的Actor

	if (ownerActor)
	{
		ownerActor->SetActorLocation(
			FVector(
					FMath::FRandRange(-1, 1) * MovementRadius,
					FMath::FRandRange(-1, 1) * MovementRadius,
					FMath::FRandRange(-1, 1) * MovementRadius)
				);
	}
}
```

+++

### 4.6 

```
//CharacterDwarf.h
UCLASS()
class MYSTUDY_API ACharacterDwarf : public ACharacter
{...}

//CharacterDwarf.cpp
ACharacterDwarf::ACharacterDwarf()
{
	PrimaryActorTick.bCanEverTick = true;	
	ConstructorHelpers::FObjectFinder<USkeletalMesh> SkeletalMeshAsset (TEXT("asset"));

	if (SkeletalMeshAsset.Object)//if (SkeletalMeshAsset.Succeeded())//效果一样		
	{
		GetMesh()->SetSkeletalMesh(SkeletalMeshAsset.Object);//设置Mesh 继承自ACharacter
		GetMesh()->SetRelativeLocation(FVector(0, 0, -60.0f));
		GetMesh()->SetRelativeRotation(FRotator(0,-90,0));
	}
	GetCapsuleComponent()->SetCapsuleHalfHeight(65.0f);//设置Capsule 继承自ACharacter
	GetCapsuleComponent()->SetCapsuleRadius(50.0f);
}

void ACharacterDwarf::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
	SetActorLocation(GetActorLocation() + GetActorForwardVector());
}
```

+++

```
//BarrackBase.h
UCLASS()
class MYSTUDY_API ABarrackBase : public AActor
{
...
public:	
	virtual void Tick(float DeltaTime) override;

	UPROPERTY(VisibleAnywhere, Category = MeshComp)
		UStaticMeshComponent* BuildMesh;	

	UPROPERTY(VisibleAnywhere, Category = MeshComp)
		UParticleSystemComponent* SpawnPoint;

	UFUNCTION()
		void SpawnUnit();
	
	UFUNCTION(BlueprintCallable, Category = SpawnTimerProperty)
		void ActorSpawnUseTimer(float LiveTime = 1.0f, bool IsLooping = false);

	FTimerHandle timeHandler;
};
```

```
//BarrackBase.cpp
ABarrackBase::ABarrackBase()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	BuildMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BuildMesh"));
	RootComponent = BuildMesh;
	SpawnPoint = CreateDefaultSubobject<UParticleSystemComponent>(TEXT("SpawnPoint"));
	SpawnPoint->SetupAttachment(GetRootComponent());

	ConstructorHelpers::FObjectFinder<UStaticMesh> MeshAsset(TEXT("StaticMesh'/..."));
	ConstructorHelpers::FObjectFinder<UParticleSystem> ParticleAsset(TEXT("..."));
	
	if (MeshAsset.Object != nullptr) 
		BuildMesh->SetStaticMesh(MeshAsset.Object);

	if (ParticleAsset.Object != nullptr) 
		SpawnPoint->SetTemplate(ParticleAsset.Object);
		
	BuildMesh->SetRelativeRotation(FRotator(0, -90, 0));
	//BuildMesh->SetRelativeScale3D(FVector(0.2f, 0.2f, 0.2f));
}

// Called when the game starts or when spawned
void ABarrackBase::BeginPlay()
{
	Super::BeginPlay();
	//GetWorld()->GetTimerManager().SetTimer(SpawnTimerHandler, this, &ABarrackBase::SpawnUnit, SpawnInterval, true); 
	ActorSpawnUseTimer(1.0f,true);
}

void ABarrackBase::SpawnUnit()
{
	GetWorld()->SpawnActor<ACharacterDwarf>(ACharacterDwarf::StaticClass(), SpawnPoint->GetComponentLocation(), SpawnPoint->GetComponentRotation() - FRotator(0,-90,0));
}

void ABarrackBase::ActorSpawnUseTimer(float LiveTime /*= 1.0f*/, bool IsLooping /*= false*/)
{
	FTimerManager &timerManager = GetWorldTimerManager();	
	timerManager.SetTimer(timeHandler, this, &ABarrackBase::SpawnUnit, LiveTime, IsLooping);
}

void ABarrackBase::EndPlay(const EEndPlayReason::Type EndPlayReason)
{  
	Super::EndPlay(EndPlayReason); 
	GetWorld()->GetTimerManager().ClearTimer(timeHandler);
} 
```

+++

# 5. Delegate And Event

### 5.1 DelegateGameMode

```
//DelegateGameMode.h
DECLARE_DELEGATE(FStandardDelegateSignature)//静态单播无参数代理类型
DECLARE_DELEGATE_OneParam(FParamDelegateSignature,FLinearColor)//单播一个参数
DECLARE_MULTICAST_DELEGATE(FMulticastDelegateSignature)//多播

UCLASS()
class MYSTUDY_API ADelegateGameMode : public AGameModeBase
{
	GENERATED_BODY()		
public:
	//定义代理对象
	FStandardDelegateSignature StandardDelegate;
	FParamDelegateSignature ParamDelegate;
	FMulticastDelegateSignature MulticastDelegate;	
};
```

+++

### 5.2 DelegateListener

```
//DelegateListener.h
UCLASS()
class MYSTUDY_API ADelegateListener : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	ADelegateListener();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;
	virtual void EndPlay(const EEndPlayReason::Type EndPlayReason);
	
public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	UPROPERTY(VisibleAnywhere)
		UPointLightComponent* PointLight;

	UFUNCTION(BlueprintCallable)
	void EnableLight();


	UFUNCTION(BlueprintCallable)
		void MySetLightColor(FLinearColor LightColor);

	UFUNCTION(BlueprintCallable)
		void MySetLightColorWithPayload(FLinearColor LightColor, bool IsEnable);

	bool IsEnable = true;
};
```

+++

```
//DelegateListener.cpp
// Sets default values
ADelegateListener::ADelegateListener()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	PointLight = CreateDefaultSubobject<UPointLightComponent>(TEXT("PointLight"));
	RootComponent = PointLight;

	PointLight->SetVisibility(false);

	//PointLight->LightColor

}

// Called when the game starts or when spawned
void ADelegateListener::BeginPlay()
{
	Super::BeginPlay();

	UWorld *world = GetWorld();
	if (world)
	{
		AGameModeBase *gm = UGameplayStatics::GetGameMode(world);

		ADelegateGameMode* delegateGM = Cast<ADelegateGameMode>(gm);
		if (delegateGM)
		{
			//delegateGM->StandardDelegate.BindUObject(this, &ADelegateListener::EnableLight);

			delegateGM->ParamDelegate.BindUObject(this, &ADelegateListener::MySetLightColor);
			
			//payload
			//delegateGM->ParamDelegate.BindUObject(this, &ADelegateListener::MySetLightColorWithPayload, IsEnable);

			delegateGM->MulticastDelegate.AddUObject(this, &ADelegateListener::EnableLight);
		}
	}

	//delegateGM->StandardDelegate.ExecuteIfBound();
}

void ADelegateListener::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
	Super::EndPlay(EndPlayReason);

	UWorld *world = GetWorld();
	if (world)
	{
		AGameModeBase *gm = UGameplayStatics::GetGameMode(world);

		ADelegateGameMode* delegateGM = Cast<ADelegateGameMode>(gm);
		if (delegateGM)
		{
			delegateGM->StandardDelegate.Unbind();

			UE_LOG(LogTemp, Warning, TEXT("Unbind"));

			delegateGM->ParamDelegate.Unbind();

			delegateGM->MulticastDelegate.Clear();
		}
	}
}

// Called every frame
void ADelegateListener::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

void ADelegateListener::EnableLight()
{
	PointLight->ToggleVisibility();
	UE_LOG(LogTemp, Warning, TEXT("EnableLight"));

}

void ADelegateListener::MySetLightColor(FLinearColor LightColor)
{
	PointLight->SetLightColor(LightColor);
}

void ADelegateListener::MySetLightColorWithPayload(FLinearColor LightColor, bool IsEnable)
{
	PointLight->SetLightColor(LightColor);
	PointLight->SetVisibility(IsEnable);
}
```

+++

### 5.3 ABoomTriggerVolume

```
//ABoomTriggerVolume.h
DECLARE_EVENT(ABoomTriggerVolume, FPlayerEnteredEvent)

UCLASS()
class MYSTUDY_API ABoomTriggerVolume : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	ABoomTriggerVolume();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	virtual void NotifyActorBeginOverlap(AActor* OtherActor) override;
	virtual void NotifyActorEndOverlap(AActor* OtherActor) override;

	UPROPERTY()
		class UBoxComponent* TriggerBox;

	//事件对象
	FPlayerEnteredEvent OnPlayerEntered;
};
```

+++

```
//BoomTriggerVolume.cpp
ABoomTriggerVolume::ABoomTriggerVolume()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	TriggerBox = CreateDefaultSubobject<UBoxComponent>(TEXT("TriggerBox"));
	TriggerBox->SetBoxExtent(FVector(200, 200, 100));
	RootComponent = TriggerBox;
}

// Called when the game starts or when spawned
void ABoomTriggerVolume::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void ABoomTriggerVolume::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

void ABoomTriggerVolume::NotifyActorBeginOverlap(AActor* OtherActor)
{
	UE_LOG(LogTemp, Warning, TEXT("BeginOverlap"));
	//OnPlayerEntered.Broadcast();
	UWorld *world = GetWorld();
	if (world)
	{
		AGameModeBase *gm = UGameplayStatics::GetGameMode(world);

		ADelegateGameMode* delegateGM = Cast<ADelegateGameMode>(gm);
		if (delegateGM)
		{
			delegateGM->StandardDelegate.ExecuteIfBound();
			UE_LOG(LogTemp, Warning, TEXT("BeginOverlap ExecuteIfBound"));

			delegateGM->ParamDelegate.ExecuteIfBound(FLinearColor::MakeRandomColor());

			//多播代理执行
			delegateGM->MulticastDelegate.Broadcast();

			OnPlayerEntered.Broadcast();
		}
	}
}

void ABoomTriggerVolume::NotifyActorEndOverlap(AActor* OtherActor)
{
	UWorld *world = GetWorld();
	if (world)
	{
		AGameModeBase *gm = UGameplayStatics::GetGameMode(world);

		ADelegateGameMode* delegateGM = Cast<ADelegateGameMode>(gm);
		if (delegateGM)
		{
			delegateGM->StandardDelegate.ExecuteIfBound();
		}
	}
}
```





