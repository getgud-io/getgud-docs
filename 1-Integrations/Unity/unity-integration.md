# Getgud Unity Integration Tutorial

## Introduction

In this tutorial, we will integrate the GetgudSDK using the [SimpleFPS project](https://github.com/RiptideNetworking/SampleFPS) as an example. You can view a full video overview of the project [here](https://www.youtube.com/watch?v=6kWNZOFcFQw&t=664s&ab_channel=TomWeiland).

We will create Client and Server Unity projects using SimpleFPS, focusing on integrating GetgudSDK into the Server Unity project. Before starting, ensure you have built the Getgud SDK by following [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-unity).

[Video Tutorial: How To Run Unity Example On Windows With Getgud SDK](https://www.youtube.com/watch?v=tyX8Oml-ReE)

[Video Tutorial: How To Run Unity Example On Linux With Getgud SDK](https://www.youtube.com/watch?v=LbeZ2FBOPTI)

## Integration Steps

### 1. Create Projects
Create "Client" and "Server" Unity projects in Unity Hub using the `Universal 3D Sample` template.

### 2. Replace Project Folders
After creating the projects, delete the `Assets`, `Packages`, and `ProjectSettings` folders in each project. Replace those folders with folders from the [Getgud Unity example](https://github.com/getgud-io/cpp-getgud-sdk-dev/tree/main/examples/unity).

### 3. Add SDK Build File
Place your Getgud SDK build file into the `Server/Assets/Plugins/GetgudSDK/` folder.
To learn how to build Getgud.io SDK you can visit [the SDK build tutorial page](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md).

### 4. Create Configuration File
Create a Getgud `config.json` file in the `Server/` folder with the following content:

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

You can also take the config file from the root of the SDK repository.

Set the environment variables:

For Linux, use the `export` command:
```bash
export GETGUD_CONFIG_PATH=/path/to/your/config.json
export GETGUD_LOG_FILE_PATH=/path/to/your/logs.txt
```

For Windows, use `setx` or the Environment Variables menu.
<b>[IMPORTANT]</b> You may want to reload your Windows instance so that env variables will be seen in your system!

### 5. Review Supported Actions
Find the list of supported actions in C# in the `Assets/Plugins/GetgudSDK/GetgudSDK_calls.cs` file.

### 6. Initialize GetgudSDK
Choose one of these methods to initialize the SDK in `Assets/Scripts/Multiplayer/NetworkManager.cs`:
- Use `GetgudSDK.Methods.InitConfPath(configFullPath)`. Make sure you specify the full file path to the config file.
- Use `GetgudSDK.Methods.Init()` if you set the `GETGUD_CONFIG_PATH` env variable.

### 7. Start Game and Match
Call `GetgudSDK.Methods.StartGame()` and `GetgudSDK.Methods.StartMatch()` with the appropriate parameters when the game and match start.

In this example, these methods are called when the server starts in `Assets/Scripts/Multiplayer/NetworkManager.cs`.

Make sure to provide your `TITLE_ID` and `PRIVATE_KEY` which you received from the Getgud.io dashboard on the StartGame method:

```csharp
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

### 8. Send Player Actions
Use the appropriate GetgudSDK commands to send player actions. In this example, we call:
- `GetgudSDK.Methods.SendSpawnAction` [here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unity/Server/Assets/Scripts/Player.cs#L157C30-L157C45)
- `GetgudSDK.Methods.SendPositionAction` [here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unity/Server/Assets/Scripts/PlayerMovement.cs#L165)


### 9. Implement Other SDK Commands
Send [other SDK commands](https://github.com/getgud-io/getgud-docs/blob/main/sdk-commands.md) in a similar way according to your needs.

### 10. Test the Integration
1. Observe the `logs.txt` to ensure that the client and server are communicating correctly.
2. Verify that actions are being sent to Getgud.io's cloud using the dashboard. [This tutorial](https://github.com/getgud-io/getgud-docs/blob/main/2-Platform/get-started-with-dashboard.md) can help you get started with Getgud.io Dashboard. 

## Conclusion

By following these steps, you should have successfully integrated the GetgudSDK into your Unity project using the SimpleFPS example.
