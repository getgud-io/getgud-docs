# Integrate Unreal Engine server with Getgud.io

1. Add the following folder structure to your UE5 Project. Make sure to create ThirdParty folder if you do not have one.

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


- bin folder - Put .dll / .so files, they represent compiled library of GetGudSDK (you can take those files from our releases OR you can build SDK yourself from source)

- include folder - Put all public headers, you can find headers [here](https://github.com/getgud-io/cpp-getgud-sdk/tree/main/include)

- lib folder - put .lib file IF you are on Windows OR if you use static library (you get .lib during build process if you are on Windows OR if you build static library)

2. Add SDK Initialization into <PROJECT_NAME>.Builds.cs file

Here is the project structure where you can find Build.cs file


```

YourProject/
├── Source/
│   ├── YourProject/
│   │   ├── YourProject.Build.cs
│   │   ├── YourProject.cpp
│   │   └── YourProject.h


```


Example of adding SDK initialization code to Build.cs file


```
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




3. [RECOMMENDED] Init SDK from your code on server start using the following command AND add SDK dispose on server stop:

Example Header: 

```
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



Example Cpp File: 
```
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



4. Follow [this tutorial](https://github.com/getgud-io/cpp-getgud-sdk/blob/main/README.md) for calling actions during events on your server


