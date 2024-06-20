# Getgud Unreal Engine Integration Tutorial

## Introduction

The `Client_Server_Getgud` project demonstrates how to integrate the GetgudSDK with Unreal Engine (UE) using a C++ based API. The project consists of two parts: a server and a client. The client sends spawn and position actions to the server, which then forwards these actions to Getgud.io's cloud.

## Prerequisites

- Unreal Engine installed
- Visual Studio installed
- GetgudSDK (Ensure you have the required credentials: `titleId` and `privateKey`)

## Project Setup

### 1. Create a New UE Project

1. Open Unreal Engine.
2. Create a new FirstPerson project with C++ and Starter Content enabled.
3. Name the project `Client_Server_Getgud`.

### 2. Integrate GetgudSDK Files

1. Upload the provided repository files to the root folder of the `Client_Server_Getgud` project.
2. Replace any existing files when prompted.
3. Delete GetgudSDK_C.cpp and GetgudSDK_C.h from `include` and `src` folders.
4. You will end up with the following file structure, notice `include` and `src` folders in ThirdParty folder:

```
Client_Server_Getgud
+---Plugins
|   \---GetgudSDK
|       |   GetgudSDK.uplugin
|       |
|       \---Source
|           \---GetgudSDKModule
|               |   GetgudSDKModule.Build.cs
|               |   GetgudSDKModule.cpp
|               |   GetgudSDKModule.h
|               |
|               \---ThirdParty
|                   +---include
|                   |
|                   \---src
|
+---Source
|   |   Client_Server_Getgud.Target.cs
|   |   Client_Server_GetgudEditor.Target.cs
|   |   Client_Server_GetgudServer.Target.cs
|   |
|   \---Client_Server_Getgud
|           Client_Server_Getgud.Build.cs
|           Client_Server_Getgud.cpp
|           Client_Server_Getgud.h
|           Client_Server_GetgudCharacter.cpp
|           Client_Server_GetgudCharacter.h
|           Client_Server_GetgudGameMode.cpp
|           Client_Server_GetgudGameMode.h
|           Client_Server_GetgudPlayerController.cpp
|           Client_Server_GetgudPlayerController.h
|           Client_Server_GetgudProjectile.cpp
|           Client_Server_GetgudProjectile.h
|           GetgudInstance.cpp
|           GetgudInstance.h
|           TP_PickUpComponent.cpp
|           TP_PickUpComponent.h
|           TP_WeaponComponent.cpp
|           TP_WeaponComponent.h
|
\---ThirdParty
    \---GetgudSDK
        \---include <----------------
```

### 3. Build the Project

1. Open the project in Visual Studio.
2. Build the project to ensure all dependencies and files are correctly integrated.

### 4. Update Project Properties

1. In Visual Studio, navigate to `Project -> Properties -> Debugging`.
2. Update the `Command Arguments` field with the following:
   ```
   "$(SolutionDir)Client_Server_Getgud.uproject" -log -server -NoGraphics -port=7777
   ```

### 5. Add Config and Log file paths

Create a `config.json` file in your project directory with the following content:

```json
{
  "streamGameURL": "https://api.staging.getgud.io/api/game_stream/send_game_packet",
  "updatePlayersURL": "https://api.staging.getgud.io/api/player_data/update_players_via_sdk",
  "sendReportsURL": "https://api.staging.getgud.io/api/report_data/send_reports",
  "throttleCheckUrl": "https://api.staging.getgud.io/api/game_stream/throttle_match_check",
  "logToFile": true,
  "logFileSizeInBytes": 10000000,
  "circularLogFile": true,
  "reportsMaxBufferSizeInBytes": 100000,
  "maxReportsToSendAtOnce": 100,
  "playersMaxBufferSizeInBytes": 100000,
  "maxPlayerUpdatesToSendAtOnce": 100,
  "maxChatMessagesToSendAtOnce": 100,
  "gameSenderSleepIntervalMilliseconds": 100,
  "apiTimeoutMilliseconds": 5000,
  "apiWaitTimeMilliseconds": 5000,
  "packetMaxSizeInBytes": 2000000,
  "actionsBufferMaxSizeInBytes": 50000000,
  "gameContainerMaxSizeInBytes": 100000000,
  "maxGames": 25,
  "maxMatchesPerGame": 50,
  "minPacketSizeForSendingInBytes": 1000000,
  "packetTimeoutInMilliseconds": 10000,
  "gameCloseGraceAfterMarkEndInMilliseconds": 20000,
  "liveGameTimeoutInMilliseconds": 60000,
  "hyperModeFeatureEnabled": false,
  "hyperModeMaxThreads": 10,
  "hyperModeAtBufferPercentage": 10,
  "hyperModeUpperPercentageBound": 90,
  "hyperModeThreadCreationStaggerMilliseconds": 100,
  "logLevel": "FULL"
}
```

Set the environment variables:
```bash
export GETGUD_CONFIG_PATH=/path/to/your/config.json
export GETGUD_LOG_FILE_PATH=/path/to/your/logs.txt
```


### 6. Run the Project

1. Run the project in debug mode using Visual Studio.
2. Open another instance of UE and load the `Client_Server_Getgud` project.
3. Run the project.

### 7. Connect to the Server

1. In the new UE instance, open the console command (`~` key).
2. Connect to the server using the command:
   ```
   open 127.0.0.1:7777
   ```

### 8. Test the Integration

1. Observe the logs to ensure that the client and server are communicating correctly.
2. Verify that actions are being sent to and from the server, and ultimately to Getgud.io's cloud.

## Detailed Code Explanation

### Configuration

In your code, configure the GetgudSDK with your credentials:

```cpp
	// Start a Game:
	g_gameGuid = GetgudSDK::StartGame(your_title_id,
		your_private_key,
		serverGuid,  // serverGuid
		gameMode,     // gameMode
		serverLocation
	);
```

### Server RPC Functions

The server handles movement and spawn actions through RPC functions.

#### Movement Handling

In `Client_Server_GetgudCharacter.h`, declare the server RPC functions:

```cpp
// Server RPC for handling movement
UFUNCTION(Server, Reliable, WithValidation)
void ServerRemoteMove(const FInputActionValue& Value);

// Server RPC for handling movement
UFUNCTION(Server, Reliable, WithValidation)
void ServerRemoteSpawn(const FInputActionValue& Value);
```

Implement these functions in `Client_Server_GetgudCharacter.cpp`:

```cpp
void AClient_Server_GetgudCharacter::ServerRemoteMove_Implementation(const FInputActionValue& Value)
{
	// Handle the movement on the server
	//Move(Value); // Call the local move function

	FRotator CameraRotation = Controller->GetControlRotation();
	FVector position = GetActorLocation();

	UE_LOG(LogTemp, Warning, TEXT("Character Position: %s"), *position.ToString());

	FDateTime Now = FDateTime::UtcNow();

	// Calculate the Unix timestamp in milliseconds
	int64 UnixTimestampMillis = (Now.GetTicks() - FDateTime(1970, 1, 1, 0, 0, 0, 0).GetTicks()) / ETimespan::TicksPerMillisecond;

	GetgudSDK::BaseActionData* outAction = nullptr;
	outAction = new GetgudSDK::PositionActionData(
		g_matchGuid, UnixTimestampMillis, g_playerGuid,
		GetgudSDK::PositionF{(float)position.X, (float)position.Y, (float)position.Z},
		GetgudSDK::RotationF{(float)CameraRotation.Pitch, (float)CameraRotation.Yaw, (float)CameraRotation.Roll});

	GetgudSDK::SendAction(outAction);

	delete outAction;
}

bool AClient_Server_GetgudCharacter::ServerRemoteMove_Validate(const FInputActionValue& Value)
{
	// Add any necessary validation here
	return true; // Assume validation passes
}
```

#### Spawn Handling

```cpp
void AClient_Server_GetgudCharacter::ServerRemoteSpawn_Implementation(const FInputActionValue& Value)
{
	// Handle the movement on the server

	FVector position = GetActorLocation();
	FRotator CameraRotation = Controller->GetControlRotation();
	UE_LOG(LogTemp, Warning, TEXT("Character Position: %s"), *position.ToString());

	FDateTime Now = FDateTime::UtcNow();

	// Calculate the Unix timestamp in milliseconds
	int64 UnixTimestampMillis = (Now.GetTicks() - FDateTime(1970, 1, 1, 0, 0, 0, 0).GetTicks()) / ETimespan::TicksPerMillisecond;

	GetgudSDK::BaseActionData* outAction = nullptr;

	outAction = new GetgudSDK::SpawnActionData(
		g_matchGuid, UnixTimestampMillis, g_playerGuid, "halls_green", 0, 100.f,
		GetgudSDK::PositionF{(float)position.X, (float)position.Y, (float)position.Z},
		GetgudSDK::RotationF{(float)CameraRotation.Pitch, (float)CameraRotation.Yaw, (float)CameraRotation.Roll});

	delete outAction;
}

bool AClient_Server_GetgudCharacter::ServerRemoteSpawn_Validate(const FInputActionValue& Value)
{
	// Add any necessary validation here
	return true; // Assume validation passes
}
```

### Initialization of SDK and clean up

The `UGetgudInstance` class handles the SDK initialization. In `UGetgudInstance.cpp`:

```cpp
void UGetgudInstance::Init()
{
    Super::Init();

    if (GetWorld()->IsNetMode(NM_DedicatedServer) || GetWorld()->IsNetMode(NM_ListenServer))
    {
        UE_LOG(LogTemp, Warning, TEXT("Getgud SDK initialized."));
        GetgudSDK::Init();
    }
}

UGetgudInstance::~UGetgudInstance()
{
    if (GetWorld() && (GetWorld()->IsNetMode(NM_DedicatedServer) || GetWorld()->IsNetMode(NM_ListenServer)))
    {
        UE_LOG(LogTemp, Warning, TEXT("Getgud SDK disposed."));
        GetgudSDK::Dispose();
    }
}

```

### Create a game and match

The `Client_Server_GetgudPlayerController` class manages the games and matches.
For example, each player has its own game and match.
In this case, a game and match are starting once a character connected:

```cpp
void AClient_Server_GetgudPlayerController::ServerRemoteGameAndMapStart_Implementation()
{

	std::string serverGuid = "us-west-1";
	std::string gameMode = "deathmatch";
	std::string serverLocation = "UK";

	// Start a Game:
	g_gameGuid = GetgudSDK::StartGame(132,
		"41e99370-b12f-11ee-89f0-4b4e9fccc950",
		serverGuid,  // serverGuid
		gameMode,     // gameMode
		serverLocation
	);

	g_matchGuid = GetgudSDK::StartMatch(g_gameGuid,
		"Knives-only",  // matchMode
		"de-dust"       // mapName
	);


	UE_LOG(LogTemp, Warning, TEXT("Getgud SDK Match started."));
}

bool AClient_Server_GetgudPlayerController::ServerRemoteGameAndMapStart_Validate()
{
	// Add any necessary validation here
	return true; // Assume validation passes
}
```

And the Game is finishing completely once the character finish the match:

```cpp
void AClient_Server_GetgudPlayerController::ServerRemoteGameEnd_Implementation()
{
	//int martResult = MarkEndGame(g_gameGuid, 36);
	GetgudSDK::MarkEndGame(g_gameGuid);

	UE_LOG(LogTemp, Warning, TEXT("Getgud SDK Game finish."));
}

bool AClient_Server_GetgudPlayerController::ServerRemoteGameEnd_Validate()
{
	// Add any necessary validation here
	return true; // Assume validation passes
}
```


### Calling RPC Functions

The RPC functions are called when the character moves or spawns. For movement:

```cpp
if (IsLocallyControlled()) // Ensure this is the local client
{
    ServerRemoteMove(Value);
}
```

For spawning:

```cpp
if (IsLocallyControlled()) // Ensure this is the local client
{
    ServerRemoteSpawn(Mesh1P->GetPosition());
}
```

## Conclusion

Following this guide, you should have successfully integrated the GetgudSDK into your UE project. By setting up the server-client architecture and implementing the necessary RPC functions, you can efficiently handle and test actions like movement and spawning within the UE environment.
