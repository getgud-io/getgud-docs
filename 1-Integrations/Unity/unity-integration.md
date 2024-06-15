# GetGud Unity Integration Tutorial

## Introduction

The `Client_Server_GetGud` project demonstrates how to integrate the GetGudSDK with Unity using a C# based API. The project is structured to handle server-client interactions, where the client sends actions like spawn and position to the server, which then communicates these actions to Getgud.io's cloud.

## Prerequisites

- Unity Hub and Unity installed
- GetGudSDK precompiled shared library
- GetGudSDK credentials: `titleId` and `privateKey`
- Basic knowledge of C# and Unity

## Project Setup

### 1. Compile the SDK

To integrate GetGudSDK with Unity, you'll first need to compile the SDK. Follow the instructions provided in the SDK documentation to compile the necessary binaries.

### 2. Create a New Unity Project

1. **Open Unity Hub:**
   - Launch Unity Hub and click on the "New" button to create a new project.

2. **Select the Template:**
   - Choose the "Universal 3D Sample" template, which provides the essential settings for a 3D project.

3. **Name and Create the Project:**
   - Name your project `Client_Server_GetGud` and specify a location on your computer. Click "Create".

### 3. Integrate GetGudSDK Files

1. **Copy Example Files:**
   - Navigate to the `examples/unity` folder within [the SDK repository](https://github.com/getgud-io/cpp-getgud-sdk-dev/tree/main/examples) and copy its contents.

2. **Paste into Project:**
   - Replace the contents of the project folder with the copied files. The Unity project should now include `Assets/GetGudSDK` among other folders.


### 4. Set Up Configuration File

1. **Create `config.json` File:**
   - In your Unity project directory, create a `config.json` file with the following content:

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

2. **Set Up Configuration Environment:**
   - Ensure your system environment variables point to the configuration file:
     ```bash
     export GETGUD_CONFIG_PATH=/path/to/your/config.json
     export GETGUD_LOG_FILE_PATH=/path/to/your/logs.txt
     ```

### 5. Update GetGudSDK Configuration

1. **Modify Path to SDK:**
   - Open the `GetgudSDK_calls.cs` file located in `Assets/GetGudSDK/Plugins` and locate line 199.
   - Update the path to point to the location of the GetGudSDK precompiled shared library.


### 6. Open and Configure Unity Project

1. **Launch Unity:**
   - Open Unity and load the `Client_Server_GetGud` project.

2. **Resolve Dependencies:**
   - Unity will automatically resolve any dependencies and might prompt you to install additional components. Allow it to do so.

3. **Verify Configuration:**
   - Ensure the `Assets/GetGudSDK` folder is present and correctly integrated with your Unity project.

### 7. Build and Verify the Project

1. **Build the Project:**
   - Go to `File -> Build Settings` and configure the build settings for your platform (e.g., Windows, macOS).
   - Click "Build and Run" to compile the project.

2. **Check for Errors:**
   - After the build completes, check the Unity console for any errors or warnings. Ensure there are no critical issues that would prevent the project from running.

### 8. Test the Integration

1. **Run the Example:**
   - In Unity, click the play button to run the project.

2. **Test Actions:**
   - Test various actions such as spawning, movement, and rotations to ensure they are communicated correctly to the server.

3. **Verify on Getgud.io:**
   - Log into your Getgud.io account and verify that the actions are recorded correctly.

### Detailed Code Explanation

### Configuration

Configure the GetGudSDK with your credentials in `GetgudSDK_calls.cs`:

```csharp
public void InitializeSDK()
{
    string titleId = "your_title_id";
    string privateKey = "your_private_key";
    GetGudSDK.Init(titleId, privateKey);
}
```

### Client-Server Communication

Implement the client-server communication for handling actions such as movement and spawn. These are typically handled through RPC functions in Unity's scripting.

#### Movement Handling

In `PlayerController.cs`, implement the movement action:

```csharp
public void SendMovement(Vector3 position, Quaternion rotation)
{
    // Construct the position action
    var positionAction = new GetGudSDK.PositionAction
    {
        Position = new GetGudSDK.Vector3F { X = position.x, Y = position.y, Z = position.z },
        Rotation = new GetGudSDK.QuaternionF { X = rotation.x, Y = rotation.y, Z = rotation.z, W = rotation.w }
    };

    // Send action to server
    GetGudSDK.SendAction(positionAction);
}
```

#### Spawn Handling

Implement the spawn action in `PlayerController.cs`:

```csharp
public void SendSpawn(Vector3 position, Quaternion rotation)
{
    // Construct the spawn action
    var spawnAction = new GetGudSDK.SpawnAction
    {
        Position = new GetGudSDK.Vector3F { X = position.x, Y = position.y, Z = position.z },
        Rotation = new GetGudSDK.QuaternionF { X = rotation.x, Y = rotation.y, Z = rotation.z, W = rotation.w }
    };

    // Send action to server
    GetGudSDK.SendAction(spawnAction);
}
```

### Initialization of SDK and Clean-Up

The `GameManager.cs` class handles the SDK initialization and clean-up:

```csharp
void Start()
{
    GetGudSDK.Init("your_title_id", "your_private_key");
    Debug.Log("GetGud SDK initialized.");
}

void OnApplicationQuit()
{
    GetGudSDK.Dispose();
    Debug.Log("GetGud SDK disposed.");
}
```

### Create and End Game

Manage game creation and termination in `GameManager.cs`:

```csharp
public void StartGame()
{
    string serverGuid = "us-west-1";
    string gameMode = "deathmatch";
    string serverLocation = "UK";

    // Start a game
    string gameGuid = GetGudSDK.StartGame("your_title_id", "your_private_key", serverGuid, gameMode, serverLocation);
    Debug.Log("Game started with GUID: " + gameGuid);
}

public void EndGame()
{
    GetGudSDK.EndGame();
    Debug.Log("Game ended.");
}
```

### Testing Actions

Ensure that actions are correctly communicated when the character moves or spawns:

```csharp
void Update()
{
    if (Input.GetKey(KeyCode.W))
    {
        SendMovement(transform.position, transform.rotation);
    }

    if (Input.GetKeyDown(KeyCode.Space))
    {
        SendSpawn(transform.position, transform.rotation);
    }
}
```
