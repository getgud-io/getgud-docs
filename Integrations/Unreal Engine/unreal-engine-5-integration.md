# Integrate Unreal Engine Server with Getgud.io

## 1. Download SDK release files or build from source

If you are using our release files:
- Download the SDK release files for UE5 from the [releases](https://github.com/getgud-io/cpp-getgud-sdk-dev/releases/)
- Unzip a release to your project's root folder as specified in step 2. You should end up with the following file structure:
    - `YourProject/ThirdParty/GetGudSDK/bin/GetGudSDK.dll`
    - `YourProject/ThirdParty/GetGudSDK/lib/GetGudSDK.lib`
    - `YourProject/ThirdParty/GetGudSDK/include/GetgudSDK_C.h`

If you are building from source follow [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/Integrations/cpp-build-instructions.md). Make sure to use our C header! 

## 2. Create Folder Structure

Add the following folder structure to your UE5 Project. Ensure you create the `ThirdParty` folder if it doesn't already exist.

```
YourProject/
├── Source/
├── Content/
├── Binaries/
├── ThirdParty/
    ├── GetGudSDK/
        ├── bin/
        ├── include/
        ├── lib/
```

- **bin folder**: Place the `.dll` / `.so` files here. These represent the compiled library of GetGudSDK (available from our releases or build the SDK yourself from the source).
- **include folder**: Place all public headers here. You can find the headers [here](https://github.com/getgud-io/cpp-getgud-sdk/tree/main/include).
- **lib folder**: Place the `.lib` file here if you are on Windows or if you use a static library. (You get the `.lib` file during the build process if you are on Windows or if you build a static library).

## 3. Add SDK Initialization to `<PROJECT_NAME>.Build.cs`

Locate the `Build.cs` file in your project structure:

```
YourProject/
├── Source/
│   ├── YourProject/
│   │   ├── YourProject.Build.cs
│   │   ├── YourProject.cpp
│   │   └── YourProject.h
```

Example of adding SDK initialization code to the `Build.cs` file:

```csharp
using System.IO;
using UnrealBuildTool;

public class MyProject : ModuleRules
{
    public MyProject(ReadOnlyTargetRules Target) : base(Target)
    {
        PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;

        PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "EnhancedInput" });

        // Setup the path to the directory containing the .lib and .dll files
        string ThirdPartyPath = Path.Combine(ModuleDirectory, "../../ThirdParty/GetGudSDK");

        PublicIncludePaths.Add(Path.Combine(ThirdPartyPath, "include"));

        // Add the .lib file for linking
        PublicAdditionalLibraries.Add(Path.Combine(ThirdPartyPath, "lib", "GetGudSDK.lib"));

        // Ensure the .dll file is copied to the Binaries/Win64 directory at runtime
        string BinariesPath = Path.Combine(ModuleDirectory, "../../Binaries/Win64", "GetGudSDK.dll");
        RuntimeDependencies.Add(BinariesPath, Path.Combine(ThirdPartyPath, "bin", "GetGudSDK.dll"));
    }
}
```

## 4. Initialize SDK in Your Code

It is recommended to initialize the SDK when your server starts and dispose it when your server stops.

Example header file:

```cpp
#pragma once

#include "GetGudSdk.h"

#include "CoreMinimal.h"
#include "Engine/GameInstance.h"
#include "MyGameInstance.generated.h"

/**
 *
 */
UCLASS()
class MYPROJECT_API UMyGameInstance : public UGameInstance
{
    GENERATED_BODY()
    
public:
    virtual void Init() override;
    virtual ~UMyGameInstance() noexcept;
};
```

Example cpp file:

```cpp
#include "MyGameInstance.h"

void UMyGameInstance::Init()
{
    Super::Init();
    GetgudSDK::Init();
}

UMyGameInstance::~UMyGameInstance()
{
    GetgudSDK::Dispose();
}
```

## 5. Add your first game

Follow [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/Integrations/C/c-integration.md#getting-started) to integrate sending events into your code.

## 6. Follow Further Tutorials

Follow [this tutorial](https://github.com/getgud-io/cpp-getgud-sdk/blob/main/README.md) for calling actions during events on your server.
