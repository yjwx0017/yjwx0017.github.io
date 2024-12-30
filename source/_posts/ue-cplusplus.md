---
title: UE C++ 笔记
date: 2024-12-30 23:53:22
tags: [虚幻引擎,UE]
categories: 日志
---

[Unreal Engine C++ Complete Guide](https://www.tomlooman.com/unreal-engine-cpp-guide/)

## 构造函数中创建对象

``` c++
// This function is only used within constructors to create new instances of our components. Outside of the constructor we use NewObject<T>();
CameraComp = CreateDefaultSubobject<UCameraComponent>("CameraComp");
// We can now safely call functions on the component
CameraComp->SetupAttachment(SpringArmComp);
```

## 构造函数外创建对象

``` c++
NewObject<T>()
```

## 生成 Actor

``` c++
GetWorld()->SpawnActor<T>()
```

## 使用 TObjectPtr

在发布的版本中，它的功能与原始指针相同。您可以继续使用原始指针，但 Epic 建议尽可能改用 `TObjectPtr`。

`TObjectPtr<T>` 仅用于标头中的成员属性，.cpp文件中的 C++ 代码继续使用原始指针，因为在函数中使用 `TObjectPtr` 没有任何好处，并且生存期较短。

``` c++
UPROPERTY(VisibleAnywhere)
TObjectPtr<UCameraComponent> CameraComp;
```

## 代码片段

``` c++
// 生成Emitter
UGameplayStatics::SpawnEmitterAttached(
    CastingEffect, Character->GetMesh(), HandSocketName, 
    FVector::ZeroVector, FRotator::ZeroRotator, EAttachLocation::SnapToTarget);
// 播放声音
UGameplayStatics::PlaySoundAtLocation(this, SoundOnStagger, GetActorLocation());
```

<!--more-->

## UWorld 和 上下文对象

UWorld 通常是您玩的关卡/世界，但在编辑器中，它可以是许多其他内容（例如，静态网格编辑器有自己的 UWorld）。很多东西都需要 UWorld，所以你经常会看到 static 函数的第一个参数是World。

UObject* WorldContextObject 可以是位于相关世界中的任何内容，例如调用此函数的角色。所以大多数时候你可以将 'this' 关键字作为第一个参数传递。


## 代码片段

``` c++
// Called to bind functionality to input
void ASCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    Super::SetupPlayerInputComponent(PlayerInputComponent);

    PlayerInputComponent->BindAxis("MoveForward", this, &ASCharacter::MoveForward);
    PlayerInputComponent->BindAxis("MoveRight", this, &ASCharacter::MoveRight);
}
```

## 前向声明

前向声明的目的是减少编译时间和类之间的依赖关系。

## 强制类型转换

`Cast<T>`

``` c++
APawn* MyPawn = GetPawn();
ASCharacter* MyCharacter = Cast<ASCharacter>(MyPawn);
if (MyCharacter) // verify the cast succeeded before calling functions
{
    // Respawn() is defined in ASCharacter, and doesn't exist in the base class APawn. Therefore we must first Cast to the appropriate class.
    MyCharacter->Respawn(); 
}
```

## 接口

``` c++
// This class does not need to be modified.
UINTERFACE(MinimalAPI)
class USGameplayInterface : public UInterface
{
    GENERATED_BODY()
};

/**
 * 
 */
class ACTIONROGUELIKE_API ISGameplayInterface
{
    GENERATED_BODY()

    // Add interface functions to this class. This is the class that will be inherited to implement this interface.
public:
    UFUNCTION(BlueprintCallable, BlueprintNativeEvent)
    void Interact(APawn* InstigatorPawn);
};

// 实现接口
UCLASS()
class ACTIONROGUELIKE_API ASItemChest : public AActor, public ISGameplayInterface // 'inherit' from interface
{
    GENERATED_BODY()

    // declared as _Implementation since we defined the function in interface as BlueprintNativeEvent
    void Interact_Implementation(APawn* InstigatorPawn); 
}

// 是否实现接口
if (MyActor->Implements<USGameplayInterface>())
{
}

// 调用接口
ISGameplayInterface::Execute_Interact(MyActor, MyParam1);
// 还有其他方法可以调用此函数，例如将 Actor 转换为接口类型并直接调用该函数。
// 但是，当接口在 Blueprint 而不是 C++ 中添加/继承到您的类时，这完全会失败，因此建议完全避免这种情况。
```

## 使用组件

如果您想在 Actor 之间共享功能，但不想使用基类，则可以使用 ActorComponent。


## 委托

Dynamic Multicast Delegates 在 Blueprint 中也称为 Event Dispatcher。

``` c++
// 定义委托类型
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(FOnAttributeChanged, AActor, InstigatorActor, USAttributeComponent, OwningComp, float, NewValue, float, Delta);

// 定义委托变量
// BlueprintAssignable，这是 Dynamic 委托的一个强大功能，可以公开给蓝图并在 EventGraph 上使用。
UPROPERTY(BlueprintAssignable, Category = "Attributes")
FOnAttributeChanged OnHealthChanged;

// 绑定委托
AttributeComp->OnHealthChanged.AddDynamic(this, &ASAICharacter::OnHealthChanged);
// UFUNCTION()
// void OnHealthChanged(AActor* InstigatorActor, USAttributeComponent* OwningComp, float NewHealth, float Delta);
// 永远不要在构造函数中绑定委托
// 应使用 AActor::PostInitializeComponents() or BeginPlay()

// 执行委托
OnHealthChanged.Broadcast(InstigatorActor, this, NewHealth, Delta);
```

由于代理是弱引用的，因此在销毁对象/Actor 时，通常不需要取消绑定代理，除非您想手动停止侦听/响应特定事件。

动态委托的性能低于非动态变体。因此，建议仅在你想将其公开给 Blueprint 时才使用此类型。

当仅在 C++ 中使用时，我们可以定义具有未指定数量的参数的委托。

``` c++
DECLARE_DELEGATE(FStreamableDelegate);

FStreamableDelegate Delegate = FStreamableDelegate::CreateUObject(this, &ASGameModeBase::OnMonsterLoaded, MonsterId, Locations[0]);
Manager->LoadPrimaryAsset(MonsterId, Bundles, Delegate);
```

## 其他

通常，您的头文件放在 Public 文件夹中，以便其他模块可以访问，而 cpp 文件位于 Private 文件夹中。不打算被其他模块直接使用的 Headers 也可以进入 Private 文件夹。

如果您不打算让其他模块依赖它，则您的主游戏模块不需要此 public/private 结构。

G 开头的类，全局可访问，例如GEngine。

FString, FName, FText

FName，本质上是哈希字符串，允许在两个 FNames 之间更快地进行比较。（FNames 在创建后不会更改），并且通常用于查找，例如骨架网格体上的 SocketNames 和 GameplayTags。

[String Handling](https://dev.epicgames.com/documentation/en-us/unreal-engine/string-handling-in-unreal-engine/?application_version=5.4)

FVector, FRotator, FTransform (FQuat)

您将主要在游戏代码中使用 FRotator，FQuat 在引擎模块之外较少使用，尽管它可以防止万向节锁定。

TArray, TMap, TSet

TSubclassOf<T>

``` c++
UPROPERTY(EditAnywhere) // Expose to Blueprint
TSubclassOf<AProjectileActor> ProjectileClass; // The class to assign in Blueprint, eg. BP_MyMagicProjectile.

FTransform SpawnTM = FTransform(ProjRotation, HandLocation);
GetWorld()->SpawnActor<AActor>(ProjectileClass, SpawnTM, SpawnParams);
```

GENERATED_BODY()

位于类和结构体的顶部

USTRUCT, UCLASS, UENUM

[UE_LOG (Logging) 打印日志](https://unrealcommunity.wiki/logging-lgpidy6i)

``` c++
// The simple logging without additional info about the context
UE_LOG(LogAI, Log, TEXT("Just a simple log print"));
// Putting actual data and numbers here is a lot more useful though!
UE_LOG(LogAI, Warning, TEXT("X went wrong in Actor %s"), *GetName()); 
```

## 函数说明符 UFUNCTION

[UFUNCTION 官方文档](https://dev.epicgames.com/documentation/en-us/unreal-engine/ufunctions-in-unreal-engine)

[UFUNCTION 说明文档](https://benui.ca/unreal/ufunction/)

如果值没有空格，则引号是可选的。例如，`someBool=true 与 someBool="true"` 的处理方式相同。同样，为了提高可读性，我建议仅在字符串类型属性周围使用引号。

``` c++
UFUNCTION(BlueprintCallable)            // 蓝图可调用
UFUNCTION(BlueprintImplementableEvent)  // 可在蓝图中实现该函数
UFUNCTION(BlueprintNativeEvent)         // C++实现，蓝图可覆盖实现 `_Implementation`后缀C++实现
UFUNCTION(meta=(ForceAsFunction))       // 强制为函数，配合BlueprintImplementableEvent
UFUNCTION(BlueprintPure)                // 纯函数（无执行引脚），默认const函数为纯函数，BlueprintPure=false
                                        // Pure 函数不会缓存其结果，因此在执行任何繁重工作时要小心，最好避免在蓝图纯函数中输出数组属性。
UFUNCTION(meta=(AutoCreateRefTerm="val1,val2")) // 未连接的引脚变量自动创建默认值

// 自动转换节点
UFUNCTION(BlueprintPure, meta=(BlueprintAutocast)) 
static EAnimal IntToAnimal(int32 Input);

UFUNCTION(meta=(BlueprintInternalUseOnly=true))  // 用于实现另一个函数或节点。它永远不会直接显示在蓝图图表中。
UFUNCTION(meta=(BlueprintProtected))             // 此函数只能在蓝图中的所属对象上调用，不能在其他实例上调用。
UFUNCTION(meta=(CallableWithoutWorldContext))    
UFUNCTION(meta=(CommutativeAssociativeBinaryOperator)) // 具有 Add Pin 按钮，可创建其他输入引脚。
UFUNCTION(SealedEvent)                           // 此函数不能在子类中覆盖。只能用于事件。
UFUNCTION(meta=(CustomStructureParam="abc"))
UFUNCTION(meta=(DefaultToSelf))
UFUNCTION(meta=(DeprecatedFunction))             // 已弃用编译警告
UFUNCTION(meta=(DeprecationMessage="abc"))       // 已弃用编译警告消息

// DeterminesOutputType & DynamicOutputParam
UFUNCTION(BlueprintCallable, Category="Utilities", meta=(WorldContext="WorldContextObject", DeterminesOutputType="ActorClass", DynamicOutputParam="OutActors"))
static void GetAllActorsOfClass(const UObject* WorldContextObject, TSubclassOf<AActor> ActorClass, TArray<AActor*>& OutActors);

// 函数的返回类型将动态更改，以匹配连接到命名参数引脚的输入。参数应为模板化类型（如 TSubClassOf<X> 或 TSoftObjectPtr<X>）

UFUNCTION(meta=(ExpandEnumAsExecs="abc"))

UFUNCTION(BlueprintCallable, meta=(ExpandEnumAsExecs="Animal"))
static void SwitchAnimalByName(FString Name, EAnimalType& Animal);
//对于 BlueprintCallable 函数，这表示应为参数使用的枚举中的每个条目创建一个输入执行引脚。该参数必须是具有 UENUM 标签的枚举类型。

UFUNCTION(BlueprintCallable, meta=(ExpandBoolAsExecs="ReturnValue"))
static bool IsAboveFreezing(float Temperature);
UFUNCTION(BlueprintCallable, meta=(ExpandBoolAsExecs="bFreezing"))
static void IsAboveFreezing(float Temperature, bool& bFreezing);

UFUNCTION(meta=(AllowPrivateAccess=true))

UFUNCTION(meta=(ArrayParm="abc"))
UFUNCTION(meta=(ArrayTypeDependentParams="abc"))
UFUNCTION(CustomThunk)

UFUNCTION(BlueprintCallable, meta=(Number="1"))
void DefaultParamInBoth(int32 Number = 99); // 被覆盖
UFUNCTION(BlueprintCallable, meta=(ComponentClass="ActorComponent"))
void DefaultTSubclassValue(TSubclassOf<UActorComponent> ComponentClass);
UFUNCTION(BlueprintCallable, meta=(RectColor="(R=0,G=1,B=0,A=1)"))
void DefaultStructValue(FLinearColor RectColor);

// 为参数指定默认值，会覆盖C++设置的默认值

UFUNCTION(CallInEditor)
void MainPosition();
// 可以通过 Details （细节） 面板中的按钮在编辑器中的选定实例上调用此函数。

UFUNCTION(Exec, meta=(OverrideNativeName="Player.SetHealth"))
// 该函数可以从游戏内控制台执行。Exec 命令仅在特定 Class 中声明时起作用。

UFUNCTION(meta=(AdvancedDisplay=2)        // 除前两个参数之外的所有参数被折叠
UFUNCTION(meta=(CompactNodeTitle="abc"))  // 紧凑显示模式显示的名称，例如+运算符
UFUNCTION(meta=(DisplayName="abc"))       // 蓝图中此节点的名称将替换为此处提供的值
UFUNCTION(meta=(ReturnDisplayName="abc")) // 返回值引脚的名称
UFUNCTION(meta=(HidePin="abc"))           // 隐藏引脚，以这种方式，每个函数只能隐藏一个 pin。
UFUNCTION(meta=(HideSelfPin))             // 隐藏Self引脚，配合DefaultToSelf
UFUNCTION(meta=(ShortToolTip="abc"))
UFUNCTION(meta=(ToolTip="abc"))           // 覆盖从代码注释自动生成的工具提示。
UFUNCTION(meta=(DevelopmentOnly))
UFUNCTION(meta=(InternalUseParam="abc"))  // 与 HidePin 类似，只能用于每个函数的一个参数。
UFUNCTION(meta=(KeyWords="abc"))          // 指定在搜索此函数时可以使用的一组关键字
UFUNCTION(meta=(Latent,LatentInfo="abc")) // 参考Delay节点实现
UFUNCTION(meta=(NativeBreakFunc))         // 指示函数的显示方式应与标准 Break Struct 节点相同。
UFUNCTION(meta=(NotBlueprintThreadSafe))
UFUNCTION(meta=(UnsafeDuringActorConstruction)) // 在 Actor 构造期间调用此函数是不安全的。
UFUNCTION(meta=(WorldContext="abc"))      // 指示哪个参数确定操作发生的 World。
UFUNCTION(meta=(MaterialParameterCollectionFunction)) // 参考SetScalarParameterValue节点实现
UFUNCTION(Server)                         // 仅在服务器上执行，`_Implementation`后缀函数中实现
UFUNCTION(Client)                         // 仅在拥有调用该函数的 Object 的客户端上执行，后缀函数中实现
UFUNCTION(Remote)                         // RPC 在连接的远程端执行。远程端可以是服务器或客户端
UFUNCTION(NetMulticast)
UFUNCTION(BlueprintAuthorityOnly)
UFUNCTION(WithValidation)                 // `_Validate`后缀函数指示是否应继续对主函数的调用。
UFUNCTION(BlueprintCosmetic)              // 此功能是装饰性的，不会在专用服务器上运行。
UFUNCTION(Reliable)                       // 仅在与 Client 或 Server 结合使用时有效。
UFUNCTION(Unreliable)                     // 仅在与 Client 或 Server 结合使用时有效。
UFUNCTION(ServiceRequest)                 // 此函数是 RPC 服务请求。这意味着 NetMulticast 和 Reliable。
UFUNCTION(ServiceResponse)                // 此函数是 RPC 服务响应。这意味着 NetMulticast 和 Reliable。
UFUNCTION(meta=(DataTablePin="abc"))      // 用于标识 DataTable Pin的元数据。
UFUNCTION(meta=(SetParam="abc"))          // 参考BlueprintSetLibrary
UFUNCTION(meta=(MapParam="abc"))          // 参考BlueprintMapLibrary
UFUNCTION(meta=(MapKeyParam="abc"))
UFUNCTION(meta=(MapValueParam="abc"))
```

## 属性说明符 UPROPERTY

[UPROPERTY 官方文档](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-uproperties)
[UPROPERTY 说明文档](https://benui.ca/unreal/uproperty/)

``` c++
UPROPERTY(VisibleAnywhere)       // 只读，细节面板（蓝图、实例），图表中不可见（需要使用 BlueprintReadOnly）。
UPROPERTY(VisibleDefaultsOnly)   // 只读，细节面板（蓝图）
UPROPERTY(VisibleInstanceOnly)   // 只读，细节面板（实例）
UPROPERTY(EditAnywhere)          // 可编辑，细节面板（蓝图、实例）
UPROPERTY(EditDefaultsOnly)      // 可编辑，细节面板（蓝图）
UPROPERTY(EditInstanceOnly)      // 可编辑，细节面板（实例）
UPROPERTY(BlueprintReadOnly)     // 可在蓝图图表中访问该属性
UPROPERTY(BlueprintReadWrite)    // 可在蓝图图表中访问该属性
UPROPERTY(BlueprintGetter="abc") // 指定属性的自定义访问函数
UPROPERTY(BlueprintSetter="abc") // 指定属性的自定义访问函数
UPROPERTY(meta=(ExposeOnSpawn=true))
UPROPERTY(Category="Animals|Birds")
UPROPERTY(meta=(DisplayName="abc"))
UPROPERTY(meta=(ToolTip="abc"))
UPROPERTY(AdvancedDisplay)
UPROPERTY(SimpleDisplay)
UPROPERTY(meta=(EditCondition="bCanFly"))
UPROPERTY(meta=(DisplayAfter="abc"))
UPROPERTY(meta=(NoResetToDefault))
UPROPERTY(SaveGame)            // 设置该标志，然后可以使用代理存档器来读/写它。
UPROPERTY(Transient)           // 临时变量，这意味着它不会被保存或加载。
UPROPERTY(Replicated)          // 在.cpp中，还需要定义一个 GetLifetimeReplicatedProps 函数
UPROPERTY(NotReplicated)       // 跳过复制。这仅适用于服务请求函数中的结构成员和参数。
UPROPERTY(ReplicatedUsing="abc") // 该函数在通过网络更新属性时执行，需要 GetLifetimeReplicatedProps
UPROPERTY(Config)              // 当前值可以保存到与类关联的 .ini 文件中，并将在创建时加载。
UPROPERTY(GlobalConfig)        // 与 Config 类似，只是你不能在子类中覆盖它。
```

## Modules 模块

默认情况下，并非每个模块都加载。使用 C++ 编程时，有时需要包含其他模块才能访问其代码。一个这样的例子是 AIModule，你必须将其添加到你希望使用它的模块的 `*.build.cs` 文件中，然后才能访问它包含的任何 AI 类。

``` c#
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "AIModule", "GameplayTasks", "UMG", "GameplayTags", "OnlineSubsystem", "DeveloperSettings" });
```

您也可以通过 .uproject 而不是构建文件包含其他模块。这是编辑器在需要时（例如创建从缺失模块派生的新 C++ 类）在 AdditionalDependencies 下自动添加模块的位置。

## 垃圾回收（内存管理）

默认情况下，垃圾回收每 60 秒进行一次，并将清理所有未引用的对象。

当调用 MyActor->DestroyActor（） 时，该 Actor 将从世界中删除，并准备从内存中清除。要正确管理“引用计数”和内存，您应该将 UPROPERTY（） 添加到 C++ 中的指针。

GC 可能需要一些时间才能启动并实际删除内存/对象。在使用 UMG 和 GetAllWidgetsOfClass 时，您可能会遇到这种情况。从视窗中删除 Widget 时，它将保留在内存中，并且仍由该函数返回，直到 GC 启动并验证所有引用均已清除。

请务必注意在运行时创建和删除的对象数量，因为垃圾回收很容易占用大量帧时间，并导致游戏过程中卡顿。需要考虑对象池等概念。

`TWeakObjPtr<T>` (UObjects)

如果我们是引用内存或对象的最后一段代码，我们会告诉引擎，我们不想保留内存或对象。当没有代码持有对它的（硬）引用时，UObjects 会自动销毁并进行垃圾回收。

``` c++
// UGameAbility derived from UObject
TWeakObjectPtr<UGameAbility> MyReferencedAbility;
UGameAbility* Ability = MyReferencedAbility.Get();
if (Ability)
{
}
```

## Class Default Object 类默认对象

是 Unreal Engine 中类的默认实例。此实例是自动创建的，用于快速实例化新实例。您也可以以其他方式使用此 CDO，以避免手动创建和维护实例。

您可以通过 `GetDefault<T>` 轻松获取 C++ 中的 CDO。您应该注意不要意外地对 CDO 进行更改，因为这会渗透到为该类创建的任何新实例中。

``` c++
// Example from: SSaveGameSubsystem.cpp (in Initialize())
const USSaveGameSettings* Settings = GetDefault<USSaveGameSettings>();

// Access default value from class
CurrentSlotName = Settings->SaveSlotName;
```

## Asserts (Debugging) 断言

``` c++
check(MyValue == 1); // treated as fatal error if statement is 'false'
check(MyActorPointer);

// convenient to break here when the pointer is nullptr we should investigate immediately
if (ensure(MyActorPointer)) // non-fatal, execution is allowed to continue, useful to encapsulate in if-statements
{
}
```

Ensure 非常适合非致命错误，并且每个会话仅触发一次。您可以使用 ensureAlways 允许 assert 在每个会话中触发多次。但是，为了您自己，请确保 assert 不在高频代码路径中，否则您将被错误报告淹没。

## Core Redirects

Core Redirects 是一种重构工具。它们允许你在通过配置文件 （.ini） 更改 C++ 后重定向几乎任何类、函数、名称等。这对于减少在 C++ 更改后更新蓝图的巨大麻烦非常有帮助。

## 打印字符串到屏幕

PrintString

[Print to Screen using C++](https://forums.unrealengine.com/t/print-to-screen-using-c/357351)

``` c++
if(GEngine)
{
    GEngine->AddOnScreenDebugMessage(-1, 15.0f, FColor::Yellow, TEXT("Some debug message!"));
}
```
