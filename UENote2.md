# 0 补充

###  0.1 定时器 和 Tick计算时间

```
void UseTimer(float LiveTime = 1.0f, bool IsLoop = false);
FTimerHandle timeHandler;

void AClass::UseTimer(float LiveTime /*= 1.0f*/, bool IsLoop /*= false*/)
{
	FTimerManager &timerManager = GetWorldTimerManager();
	//定时器
	timerManager.SetTimer(timeHandler, this, &AClass::Func, LiveTime, IsLoop);
}
GetWorld()->GetTimerManager().ClearTimer(timeHandler);//清空定时器
```

+++

Tick 计算时间

```
float Seconds;//秒
int32 TimeScale; //时间倍率

void YourClass::Tick(float DeltaTime) {  
	Super::Tick(DeltaTime); 
 	Seconds += (DeltaTime * TimeScale); 
 	if (Seconds > 60) {   //大于60秒，秒置为0，分钟+1
 		Seconds -= 60;
  		Minutes++;   
  		if (Minutes > 60) {   
        	Minutes -= 60;    
        	Hours++;   
        }  
	}
}
```

+++

### 0.2 *获取场景中物体

```
UWorld *world = GetWorld();
if (world){
	UWorld *world = GetWorld();
	AGameModeBase *gm = UGameplayStatics::GetGameMode(world);
	ADelegateGM* delegateGM = Cast<ADelegateGM>(gm);//Cast转换
	
	TArray<AActor*> Array; //Array.Num()
    UGameplayStatics::GetAllActorsOfClass(world, Yourclass::StaticClass(), ); 
}
```

```
//关卡里的迭代器 TActorIterator
for (TActorIterator<AActor> It(GetWorld(), AActor::StaticClass()); It; ++It)
{
	AActor* actor = *It;
    
	//ImplementsInterface(UClass)  判断actor是否实现某接口
	if (actor->GetClass()->ImplementsInterface(UITeamInterface::StaticClass()))
	{}
	
	ITeamInterface* teamInterface = Cast<ITeamInterface>(actor);//actor也能转换成接口..
	if (teamInterface)
	{
		Count++;
	}
}
```



+++

### 0.3 SpawnActor And NewObject

```
MyActor = GetWorld()->SpawnActor<AOneActor>(AOneActor::StaticClass(), SpawnTransform);	
AActor* ownerActor = GetOwner();//得到该组件上的Actor
MyActor->SetLifeSpan(3.0f);//可以设置LifeSpan
if (MyActor != nullptr)
{
	MyActor->Destroy();
	MyActor = nullptr;
}

//UBoxComponent* TriggerBox; //Trigger
//NotifyActorBeginOverlap(AActor* OtherActor)//Actor进入triggger
//NotifyActorEndOverlap(AActor* OtherActor)//Actor离开trigger

NewObject<Class>(this, Class::StaticClass()); 
RandomCom->RegisterComponent(); //组件注册，会渲染组件
```

### 0.4 引用资源

```
//ConstructorHelper 只能在类的构造方法中才可以使用
StaticLoadObject(UStaticMesh::StaticClass(), nullptr, *ObjRefFString);
```

+++

### 0.5 创造一个点

```
UPROPERTY(EditAnywhere, meta = (MakeEditWidget = true))
	FVector TargetLocation;
	
UPROPERTY(EditAnywhere, meta = (AllowPrivateAccess = "true")) //privite属性公开
```

+++

### 0.6 RotatingMovementComponent

```
URotatingMovementComponent* Rotating;
Rotating = CreateDefaultSubobject<URotatingMovementComponent>(TEXT("Rotating"));
Rotating->RotationRate = FRotator(0, 10, 0);
```

+++

### 0.7 静态网格设置碰撞模式

```
UStaticMeshComponent* Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
Mesh->SetCollisionProfileName(TEXT("OverlapAll"));
```

+++

### 0.8 广告牌辅助表示物体场景位置

```
//辅助标识位置，一个头
CreateDefaultSubobject<UBillboardComponent>(TEXT("Billboard")); 
```

![TE13IJ`TI%$5$GG`KM5F4SN](C:\Users\1\Desktop\TE13IJ`TI%$5$GG`KM5F4SN.png)

+++

### 0.9 随机向量

```
UKismetMathLibrary::RandomUnitVector() //返回半径为1的球的一点
//随机种子
FRandomStream RandomStream; 
UKismetMathLibrary::RandomUnitVectorFromStream(RandomStream)
```

+++

### 0.10 得到子组件

```
TArray<USceneComponent*> childs;  
DefaultSceneRoot->GetChildrenComponents(true, childs);   
for (auto& comp : childs)
	comp->DestroyComponent(true);//销毁子组件
```

+++

### 0.11 添加组件

```
//NewObject  AttachToComponent
UStaticMeshComponent Comp = NewObject<UStaticMeshComponent>(this);
Comp->RegisterComponent();	//NewObject需要注册
Comp->AttachToComponent(BaseWall, FAttachmentTransformRules::KeepRelativeTransform);

//CreateDefaultSubobject
USceneComponent* Root = CreateDefaultSubobject<USceneComponent>(TEXT("Root"));
RootComponent = Root;
//Root->SetupAttachment(RootComponent);效果同上
```

+++

### 0.12 FindLookAtRotation

```
UKismetMathLibrary::FindLookAtRotation(StartLocation(), TargetLocation)//return FRotator
```

+++

### 0.13 ThirdPersonCharacter的创建

```
//继承Pawn 只有一个场景组件，Character有胶囊碰撞体，箭头，Mesh（骨骼Mesh）
	
USpringArmComponent* CameraBoom;
UCameraComponent* FollowCamera;
float BaseTurnRate;
float BaseLookUpRate;

//控制移动轴 Don't rotate when the controller rotates. Let that just affect the camera.
bUseControllerRotationPitch = false;
bUseControllerRotationYaw = false;
bUseControllerRotationRoll = false;

// Configure character movement
// Character moves in the direction of input...	
GetCharacterMovement()->bOrientRotationToMovement = true; 
 // ...at this rotation rate
GetCharacterMovement()->RotationRate = FRotator(0.0f, 540.0f, 0.0f);
GetCharacterMovement()->JumpZVelocity = 600.f;
GetCharacterMovement()->AirControl = 0.2f;

// Create a camera boom (pulls in towards the player if there is a collision)
CameraBoom = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraBoom"));
CameraBoom->SetupAttachment(RootComponent);
// The camera follows at this distance behind the character	
CameraBoom->TargetArmLength = 300.0f;
// Rotate the arm based on the controller
CameraBoom->bUsePawnControlRotation = true; 

// Create a follow camera
FollowCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FollowCamera"));
// Attach the camera to the end of the boom and let the boom adjust to match the controller orientation
FollowCamera->SetupAttachment(CameraBoom, USpringArmComponent::SocketName); 
// Camera does not rotate relative to arm
FollowCamera->bUsePawnControlRotation = false;
	

void AThirdPersonCharacter::TurnAtRate(float Rate)
{
	// calculate delta for this frame from the rate information
	AddControllerYawInput(Rate * BaseTurnRate * GetWorld()->GetDeltaSeconds());
}

void AThirdPersonCharacter::MoveForward(float Value)
{
	if ((Controller != NULL) && (Value != 0.0f))
	{
		// find out which way is forward
		const FRotator Rotation = Controller->GetControlRotation();
		const FRotator YawRotation(0, Rotation.Yaw, 0);

		// get forward vector
		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
		// get right vector 
		//const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);
		AddMovementInput(Direction, Value);
	}
}
```

+++

### 0.14* BlueprintImplementableEvent And  BlueprintNativeEvent

```
//BlueprintImplementableEvent关键字, cpp中可以不实现，让蓝图实现，c++可以调用
UFUNCTION(BlueprintCallable, BlueprintImplementableEvent)
	void OnRaycastHit();

//BlueprintNativeEvent 本地需要实现函数为 Fun_Implementation，一个默认方法
//子类蓝图可以使用该方法，也可以添加新功能或重写
UFUNCTION(BlueprintCallable,BlueprintNativeEvent)
	void DoSingleTrace();

//需要加后缀 _Implementation
void ARayCastCamera::DoSingleTrace_Implementation(){}
```

+++

### 0.15 Actor里的方法

```
//actor中
AddActorLocalOffset(FVector(0, 0, 0)); //修改位置
AddActorLocalRotation(FRotator(0, 90, 0)); //修改旋转
SetActorEnableCollision(true);
//StaticMeshActor
GetStaticMeshComponent()->SetGenerateOverlapEvents(true);
GetStaticMeshComponent()->SetMobility(EComponentMobility::Movable);
```

+++

### 0.16 类继承DefalutPawn会有移动组件，Pawn则没有



+++

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

### 1.3 屏幕打印和控制台输出

```
//向控制台输出，~~查看控制台输出
GetWorld()->GetFirstPlayerController()->ClientMessage(TEXT("hello"));
//屏幕打印
GEngine->AddOnScreenDebugMessage(-1, 10.0f, FColor::Red, TEXT("output to screen"));

GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Green, FString::Printf(TEXT("%s actor implement TeamInterface"), *actor->GetName()));
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

------

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

------

```
//ConditionalBeginDestroy() 会调用 BeginDestroy，主动销毁
UserProfile->ConditionalBeginDestroy();

//强制进行垃圾回收
GEngine->ForceGarbageCollection(true);
```

------



# 3.Memory And Smart Pointer

### 3.1 声明

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

------

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

------

### 3.3 弱指针

```
//弱指针需要先用共享指针
GameDataPtr = MakeShareable(new FGameData);
GameDataWeakPtr = GameDataPtr;

//弱指针 调用指向对象的成员函数
GameDataWeakPtr.Pin()->GetNumber()

GameDataWeakPtr.IsValid()//判断指针是否有效
```

------



# 4. Actor And Components

```
Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BaseMeshComponent"));

RootComponent = Mesh;
//效果同上，为根组件添加组件
//Mesh->SetupAttachment(GetRootComponent());
//Mesh->SetupAttachment(RootComponent);

//构造方法可用
ConstructorHelpers::FObjectFinder<UStaticMesh> MeshAsset = 	(TEXT("Asset..."));
//auto MeshAsset = ConstructorHelpers::FObjectFinder<UStaticMesh>(TEXT(""));
//ConstructorHelpers::FObjectFinder<UStaticMesh> MeshAsset(TEXT(""));
UStaticMesh Temp = MeshAsset.Object;//接收后可在其他地方用Temp

if(MeshAsset.Object != nullptr)//if (MeshAsset.Succeeded())//效果相同	
	Mesh->SetStaticMesh(MeshAsset.Object);
```

+++



# 5. Delegate And Event

```
//声明 定义。 绑定，解绑； 执行。
DECLARE_DELEGATE(FStandardDelegateSignature)//静态单播无参数代理类型
DECLARE_DELEGATE_OneParam(FParamDelegateSignature,FLinearColor)//单播一个参数
DECLARE_MULTICAST_DELEGATE(FMulticastDelegateSignature)//多播 
//还需声明定义 FStandardDelegateSignature StandardDelegate;

//在AListener类中得到 声明代理的类 并绑定方法 
//通过代理调用，绑定函数需要标记UFUNCTION宏
delegate->StandardDelegate.BindUObject(this, &AListener::EnableLight);
delegate->ParamDelegate.BindUObject(this, &AListener::MySetLightColor);
delegate->MulticastDelegate.AddUObject(this, &AListener::EnableLight);

//在AListener绑定类中解绑
delegate->ParamDelegate.Unbind();
delegate->MulticastDelegate.Clear();//多播

//第一个参数类名，在该类中执行
DECLARE_EVENT(ATriggerEvent, FPlayerEnteredEvent)//声明自定义事件，也需定义
FPlayerEnteredEvent OnPlayerEntered;	//事件对象

//在其他类Other中得到 定义自定义事件的类 并绑定该类的方法
triggerEvent->OnPlayerEntered.AddUObject(this, &Other::func);

delegate->StandardDelegate.ExecuteIfBound();
delegate->ParamDelegate.ExecuteIfBound(FLinearColor::MakeRandomColor());//一参
delegate->MulticastDelegate.Broadcast();//多播代理执行
OnPlayerEntered.Broadcast();//事件执行func()
```

+++

# 6. Input映射


```
//控制移动轴 z y,可放在构造中，整齐。。
bUseControllerRotationYaw = true;
bUseControllerRotationPitch = true;

//InputAxis 轴映射
FInputAxisKeyMapping ForwardKeyW("Forward", EKeys::W, 1.0f);
FInputAxisKeyMapping ForwardKeyS("Forward", EKeys::S, -1.0f);
//InputAction 动作映射
FInputActionKeyMapping FireKey("Fire", EKeys::LeftShift, 0, 0, 0, 0);//0表示组合键

GetWorld()->GetFirstPlayerController()->PlayerInput->AddAxisMapping(ForwardKeyW);
GetWorld()->GetFirstPlayerController()->PlayerInput->AddAxisMapping(ForwardKeyS);
GetWorld()->GetFirstPlayerController()->PlayerInput->AddActionMapping(FireKey);

//给Input事件绑定方法
PlayerInputComponent->BindAxis("Forward", this, &AWarrior::MoveForward);
PlayerInputComponent->BindAction("Fire", IE_Pressed, this, &AWarrior::OnFire);
```

+++

# 7.RayCast

```
UPROPERTY(EditAnywhere, Category = CollisionPro)
	float SingleTraceLineLength;
	
FHitResult hitResult;
const TArray<AActor*> ignoreActors;
//终点 = 当前向量 + 当前前方的单位向量 * 自定义倍数
FVector end = GetActorLocation() + GetActorForwardVector() * SingleTraceLineLength;
bool isHit = UKismetSystemLibrary::LineTraceSingle(GetWorld(), GetActorLocation(), end, UEngineTypes::ConvertToTraceType(ECC_Visibility), false, ignoreActors, EDrawDebugTrace::ForDuration, hitResult, true);
//画一条线
DrawDebugLine(GetWorld(), GetActorLocation(), end, FColor::Red);
```

+++

# 8*.Interface

### 8.1

```
接口方法公开给蓝图，用一种就行
1.UINTERFACE(MinimalAPI, meta = (CannotImplementInterfaceInBlueprint))
2.UFUNCTION(BlueprintImplementableEvents,BlueprintCallable)

//UINTERFACE(MinimalAPI)
1.UINTERFACE(MinimalAPI, meta = (CannotImplementInterfaceInBlueprint))
class UKillableInterface : public UInterface //**UKillableInterface
{
	GENERATED_BODY()
};
class MYSTUDY_API IKillableInterface //**IKillableInterface
{
	GENERATED_BODY()
	
public:
	//=0表示 纯虚函数。 抽象类不能被实例化，需在子类实现
	virtual int32 GetTeamNum() const = 0;
	
	2.UFUNCTION(BlueprintCallable)
		virtual void Die();
};

void IKillableInterface::Die(){}
```

+++

### 8.2 接口继承

```
UINTERFACE(MinimalAPI, meta = (CannotImplementInterfaceInBlueprint))
class UUndeadInterface : public UKillableInterface	//***注意区别
{
	GENERATED_BODY()
};
class MYSTUDY_API IUndeadInterface : public IKillableInterface	//***
{
	GENERATED_BODY()
public:	
	virtual void Die() override;//可以重写
	//放逐,用来杀死Undead
	UFUNCTION(BlueprintCallable)
		virtual void Banish();
};

void IUndeadInterface::Banish()
{
	AActor* self = Cast<AActor>(this);//** 得到实现该接口的Actor
	if (self)
	{
		self->Destroy();
	}
}
```

+++

### 8.3BlueprintNativeEvent

```
...
class MYSTUDY_API IOpenableInterface
{	...
	UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = Openable)
		void Open();
};
```

```
UCLASS()
class MYSTUDY_API ADoor : public AStaticMeshActor, public IOpenableInterface
{	...	
	UFUNCTION()
	//注意重写方法格式 Func_Implementation()
		virtual void Open_Implementation() override;
};

void ADoor::Open_Implementation(){}

```

```
//其他类中可以执行接口的Open方法(绑定该接口的类)
//判断DoorActor是否实现该接口
if (DoorActor->GetClass()->ImplementsInterface(UOpenableInterface::StaticClass()))
{
	IOpenableInterface::Execute_Open(DoorActor);
}
```



```
class MYSTUDY_API AInteractingPawn : public ADefaultPawn
{	...	
	void TryInteract();
};

void AInteractingPawn::TryInteract()
{
	//获取Pawn内的控制器，看是不是一个PlayerController
	//Pawn是有可能没有Controller或者是AIController的
	APlayerController *PC = Cast<APlayerController>(Controller);
	if (PC)
	{
		//获取跟PC绑定的相机控制类，这里是为了获取Ray trace的起终点
		APlayerCameraManager* camManager = PC->PlayerCameraManager;

		FVector startLoc = camManager->GetCameraLocation();
		FVector endLoc = startLoc + (camManager->GetActorForwardVector() * 100);

		//这里展示了如何使用Ray Cast 一族函数的一般典型形式
		//先声明一个FHitResult
		FHitResult hitResult;

		//使用按物体类型的ray cast
		//声明一个FCollisionObjectQueryParams
		FCollisionObjectQueryParams objectQueryParams(FCollisionObjectQueryParams::AllObjects);
		FCollisionShape shape = FCollisionShape::MakeSphere(25.0f);
		FCollisionQueryParams queryParams(FName("Interaction"), true, this);

		GetWorld()->SweepSingleByObjectType(hitResult, startLoc, endLoc, FQuat::Identity, objectQueryParams, shape, queryParams);
		if (hitResult.Actor != nullptr)
		{
			if (hitResult.Actor->GetClass()->ImplementsInterface(UInteractableInterface::StaticClass()))
			{
				if (IInteractableInterface::Execute_CanInteract(hitResult.Actor.Get()))
				{
					IInteractableInterface::Execute_PerformInteract(hitResult.Actor.Get());
				}
			}
		}
	}

}
```

































