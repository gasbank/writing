# UE5 정복

UE5 배운 내용들 메모로 남기자.

## 1일차 (2023년 3월 21일 20:00 ~ 22:00)

1. 새 프로젝트 만들기 - 이름: `StartMac`, `Empty` 템플릿, `C++`
2. 콘텐츠 생성: `레벨`, 이름: `GameMap`
3. `GameMap` 열기
4. C++ 버전의 게임 모드 베이스 만들기 - `MyGameModeBase` 클래스 타입은 정하지 않은 상태로 두기
5. 월드 세팅 창 열고 게임모드 오버라이드를 `MyGameModeBase`로 지정
6. `MyGameModeBase.h` 코드 편집 - 코드 중 `STARTMAC_API` 부분은 각자의 프로젝트명에 맞게 변경해야 함
   ```c++
   UCLASS()
    class STARTMAC_API AMyGameModeBase : public AGameModeBase
    {
        GENERATED_BODY()
    
    public:
        AMyGameModeBase();
    
    };
   ```
7. `MyGameModeBase.cpp` 코드 편집
    ```c++
    AMyGameModeBase::AMyGameModeBase()
    {
        // set default pawn class to our Blueprinted character
        static ConstructorHelpers::FClassFinder<APawn> PlayerPawnBaseBPClass(TEXT("/Game/BP_ThirdPersonCharacter"));
        if (PlayerPawnBaseBPClass.Class != NULL)
        {
            DefaultPawnClass = PlayerPawnBaseBPClass.Class;
        }
    }
    ```
8. 에디터 실행하면 시 `BP_ThirdPersonCharacter` 찾을 수 없다는 경고창 나온다. 정상.
9. 블루프린트 클래스 추가 - 종류: `캐릭터`, 이름: `BP_ThirdPersonCharacter`
10. C++ 클래스 추가 - 종류: `캐릭터`, 이름: `MyGameCharacter`
11. `BP_ThirdPersonCharacter` 애셋 열고 상단 메뉴 `파일` - `블루프린트 부모변경` - `MyGameCharacter`
12. `UnrealEngine\Templates\TemplateResources\High\Characters\Content\Mannequins\Meshes\SKM_Quinn.uasset` 파일을 내 프로젝트 콘텐츠 폴더로 복사
13. `BP_ThirdPersonCharacter` 애셋의 최상위 컴포넌트 선택하고 디테일 패널에서 `Skeletal Mesh Asset`을 `SKM_Quinn`으로 지정
14. `MyGameCharacter.h` 코드 편집 - `GENERATED_BODY()` 아래에 다음 코드 삽입
    ```c++
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = MyInput, meta = (AllowPrivateAccess = "true"))
    class UInputMappingContext* DefaultMappingContext;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = MyInput, meta = (AllowPrivateAccess = "true"))
    class UInputAction* JumpAction;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = MyInput, meta = (AllowPrivateAccess = "true"))
    class UInputAction* MoveAction;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = MyInput, meta = (AllowPrivateAccess = "true"))
    class UInputAction* LookAction;
    ```
15. `MyGameCharacter.h` 코드 편집 - `protected:` 아래에 다음 코드 삽입
    ```c++
    void Move(const FInputActionValue& Value);
    void Look(const FInputActionValue& Value);
    ```
16. 빌드 해 보면 여러 가지 빌드 오류들이 날 것
17. `StartMac.Build.cs` 파일 열어 `PublicDependencyModuleNames` 변수에 `EnhancedInput` 항목 추가
18. `MyGameCharacter.cpp` 코드 편집 - 헤더 파일 맨 뒤에 추가
    ```c++
    #include "EnhancedInputComponent.h"
    #include "EnhancedInputSubsystems.h"
    ```
19. `MyGameCharacter.h` 코드 편집 - `#include "MyGameCharacter.generated.h"` 줄 위에 추가
    ```c++
    #include "InputActionValue.h"
    ```
20. 빌드 정상적으로 완료되는지 확인
21. `MyGameCharacter.cpp` 코드 편집. 캐릭터 조작 입력 바인딩의 실질적인 구현.
    ```c++
    void AMyGameCharacter::BeginPlay()
    {
        Super::BeginPlay();

        if (const APlayerController* PlayerController = Cast<APlayerController>(Controller))
        {
            if (UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PlayerController->GetLocalPlayer()))
            {
                Subsystem->AddMappingContext(DefaultMappingContext, 0);
            }
        }
    }

    void AMyGameCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
    {
        Super::SetupPlayerInputComponent(PlayerInputComponent);

        if (UEnhancedInputComponent* EnhancedInputComponent = CastChecked<UEnhancedInputComponent>(PlayerInputComponent))
        {
            EnhancedInputComponent->BindAction(JumpAction, ETriggerEvent::Triggered, this, &ACharacter::Jump);
            EnhancedInputComponent->BindAction(JumpAction, ETriggerEvent::Completed, this, &ACharacter::StopJumping);

            EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMyGameCharacter::Move);
            EnhancedInputComponent->BindAction(LookAction, ETriggerEvent::Triggered, this, &AMyGameCharacter::Look);
        }
    }

    void AMyGameCharacter::Move(const FInputActionValue& Value)
    {
        const FVector2D MovementVector = Value.Get<FVector2D>();

        if (Controller != nullptr)
        {
            const FRotator Rotation = Controller->GetControlRotation();
            const FRotator YawRotation(0, Rotation.Yaw, 0);

            const FVector ForwardDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
            const FVector RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);

            AddMovementInput(ForwardDirection, MovementVector.Y);
            AddMovementInput(RightDirection, MovementVector.X);
        }
    }

    void AMyGameCharacter::Look(const FInputActionValue& Value)
    {
        const FVector2D LookAxisVector = Value.Get<FVector2D>();

        if (Controller != nullptr)
        {
            AddControllerYawInput(LookAxisVector.X);
            AddControllerPitchInput(LookAxisVector.Y);
        }
    }
    ```
22. 콘텐츠 생성: `입력` - `입력 매핑 컨텍스트` 1개, 이름: `IMC_Default`
23. 콘텐츠 생성: `입력` - `입력 액션` 3개, 이름 `IA_Jump`, `IA_Move`, `IA_Look`
24. `편집` - `프로젝트 세팅...` - `프로젝트` - `맵 & 모드`에 들어가서 `에디터 시작 맵`, `게임 기본 맵`을 `GameMap`으로 변경
25. 카메라를 제대로 작동하도록 해 보자.
26. `MyGameCharacter.h` 파일에 프로퍼티 추가
    ```c++
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = MyCamera, meta = (AllowPrivateAccess = "true"))
	class USpringArmComponent* CameraBoom;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = MyCamera, meta = (AllowPrivateAccess = "true"))
	class UCameraComponent* FollowCamera;
    ```
27. `MyGameCharacter.cpp` 파일 생성자에 코드 삽입
    ```c++
    // controller 회전 시에 캐릭터도 회전하게 하지 않는다. 카메라만 회전하면 된다.
	bUseControllerRotationPitch = false;
	bUseControllerRotationYaw = false;
	bUseControllerRotationRoll = false;

	GetCharacterMovement()->bOrientRotationToMovement = true;
	GetCharacterMovement()->RotationRate = FRotator(0.0f, 500.0f, 0.0f);

	GetCharacterMovement()->JumpZVelocity = 700.f;
	GetCharacterMovement()->AirControl = 0.35f;
	GetCharacterMovement()->MaxWalkSpeed = 500.f;
	GetCharacterMovement()->MinAnalogWalkSpeed = 20.f;
	GetCharacterMovement()->BrakingDecelerationWalking = 2000.f;

	CameraBoom = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraBoom"));
	CameraBoom->SetupAttachment(RootComponent);
	CameraBoom->TargetArmLength = 400.0f;
	CameraBoom->bUsePawnControlRotation = true;

	FollowCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FollowCamera"));
	FollowCamera->SetupAttachment(CameraBoom, USpringArmComponent::SocketName);
	FollowCamera->bUsePawnControlRotation = false;
    ```
28. 카메라 제어도 이제 된다.
