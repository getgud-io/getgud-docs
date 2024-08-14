# Getgud Unity Integration Tutorial

## Introduction

In this tutorial, we will integrate the GetgudSDK using the [SimpleFPS project](https://github.com/RiptideNetworking/SampleFPS) as an example. You can view a full video overview of the project [here](https://www.youtube.com/watch?v=6kWNZOFcFQw&t=664s&ab_channel=TomWeiland).

We will create Client and Server Unity projects using SimpleFPS, focusing on integrating GetgudSDK into the Server Unity project. Before starting, ensure you have built the Getgud SDK by following [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-unity).


[VIDEO: How To Run Unity Example On Windows With Getgud SDK
](https://www.youtube.com/watch?v=tyX8Oml-ReE)

[VIDEO: How To Run Unity Example On Linux With Getgud SDK
](https://www.youtube.com/watch?v=LbeZ2FBOPTI)

## Integration

### 1. Create "Client" and "Server" Unity projects in Unity Hub using the `Universal 3D Sample` template.
   
### 2. After creating the projects, delete the `Assets`, `Packages`, and `ProjectSettings` folders in each project. 
Replace those folders  with folders from the [Getgud Unity example](https://github.com/getgud-io/cpp-getgud-sdk-dev/tree/main/examples/unity).

### 3. Place your Getgud SDK build file into the `Server/Assets/Plugins/GetgudSDK/` folder.

### 4. Create a Getgud `config.json` file in the `Server/` folder. 

Paste the following default configuration into `config.json`:

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
  "maxChatMessagesToSendAtOnce": 100,
  "playersMaxBufferSizeInBytes": 100000,
  "maxPlayerUpdatesToSendAtOnce": 100,
  "gameSenderSleepIntervalMilliseconds": 100,
  "apiTimeoutMilliseconds": 5000,
  "apiWaitTimeMilliseconds": 5000,
  "packetMaxSizeInBytes": 2000000,
  "actionsBufferMaxSizeInBytes": 50000000,
  "gameContainerMaxSizeInBytes": 100000000,
  "maxGames": 25,
  "maxMatchesPerGame": 50,
  "minPacketSizeForSendingInBytes": 1000000,
  "packetTimeoutInMilliseconds": 100000,
  "gameCloseGraceAfterMarkEndInMilliseconds": 20000,
  "liveGameTimeoutInMilliseconds": 100000,
  "hyperModeFeatureEnabled": false,
  "hyperModeMaxThreads": 10,
  "hyperModeAtBufferPercentage": 10,
  "hyperModeUpperPercentageBound": 90,
  "hyperModeThreadCreationStaggerMilliseconds": 100,
  "logLevel": "FULL"
}
```

You can also take config file from the root of SDK repository. 

Now set the environment variables. 

Use `export` command on Linux:
```bash
export GETGUD_CONFIG_PATH=/path/to/your/config.json
export GETGUD_LOG_FILE_PATH=/path/to/your/logs.txt
```

Use `setx` /  Environment Variables menu on Windows.

### 5. Find the list of supported actions in C# in the `Assets/Plugins/GetgudSDK/GetgudSDK_calls.cs` file.

### 6. Initialize GetgudSDK.
1. You can use `GetgudSDK.Methods.InitConfPath(configFullPath)`. Make sure you specify full file path to config file.
2. You can use `GetgudSDK.Methods.Init()` if you set `GETGUD_CONFIG_PATH` env variable.
3.    Call one of these methods at server start in `Assets/Scripts/Multiplayer/NetworkManager.cs`. 

### 7. When the game and match start, call `GetgudSDK.Methods.StartGame()` and `GetgudSDK.Methods.StartMatch()` with the appropriate parameters.

In this example, these methods are called when the server starts in `Assets/Scripts/Multiplayer/NetworkManager.cs`.

Make sure to provide your `TITLE_ID` and `PRIVATE_KEY` which you received from Getgud.io dashboard on StartGame method.
```
  StartGameInfo gameInfo = new StartGameInfo
        {
            TitleId = 219, // Use your actual TitleId
            PrivateKey = "private-key", // Use your actual private key
            ServerGuid = "example-server-guid",
            GameMode = "example-game-mode",
            ServerLocation = "example-server-location"
        };

        int gameResult = GetgudSDK.Methods.StartGame(gameInfo, out string gameGuid);
```

### 9. To send player actions, use the appropriate GetgudSDK commands. 

In this example, we call `GetgudSDK.Methods.SendSpawnAction` and `GetgudSDK.Methods.SendPositionAction`. The spawn action is called [here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unity/Server/Assets/Scripts/Player.cs#L157C30-L157C45), and the position action is called [here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unity/Server/Assets/Scripts/PlayerMovement.cs#L165).

### 10. Integrate angle coordinates properly

Getgud requires you to send your angle coordinates in a [specific way](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/getgud-sdk-angles-tutorial.md). We integrated required transformations to send angles to Getgud, [see here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unity/Server/Assets/Scripts/PlayerMovement.cs#L151).

    ```
     // Neccessary adjustments for GetgudSDK around angles
     // See getgud docs
     // Swap pitch and yaw as required by the SDK
     float temp = pitch;
     pitch = yaw;
     yaw = temp;

     // Getgud most upward yaw angle is -89, most down yaw angle is 89.
     // we need to change sign in unity
     yaw = -yaw;

     // same with pitch. we need to reverse unity pitch to match Getgud format
     pitch = -pitch;
    ```

### 11. Send [other SDK commands](https://github.com/getgud-io/getgud-docs/blob/main/sdk-commands.md) in a similar way according to your needs.

### 12. Test the integration.
1. Observe the `logs.txt` to ensure that the client and server are communicating correctly.
2. Verify that actions are being sent to Getgud.io's cloud using dashboard.
