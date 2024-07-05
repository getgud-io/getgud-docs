# Getgud Unity Integration Tutorial

## Introduction

In this tutorial, we will integrate the GetgudSDK using the [SimpleFPS project](https://github.com/RiptideNetworking/SampleFPS) as an example. You can view a full video overview of the project [here](https://www.youtube.com/watch?v=6kWNZOFcFQw&t=664s&ab_channel=TomWeiland).

We will create Client and Server Unity projects using SimpleFPS, focusing on integrating GetgudSDK into the Server Unity project. Before starting, ensure you have built the Getgud SDK by following [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-unity).

## Integration

1. Create "Client" and "Server" Unity projects in Unity Hub using the `Universal 3D Sample` template.
   
2. After creating the projects, delete the `Assets`, `Packages`, and `ProjectSettings` folders in each project. Replace them with folders from the [Getgud Unity example](https://github.com/getgud-io/cpp-getgud-sdk-dev/tree/main/examples/unity).

3. Place your Getgud SDK build file into the `Server/Assets/Plugins/GetgudSDK/` folder.

4. Create a Getgud `config.json` file in the `Server/` folder. Paste the following default configuration into `config.json`:

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

5. Find the list of supported actions in C# in the `Assets/Plugins/GetgudSDK/GetgudSDK_calls.cs` file.

6. Initialize GetgudSDK using `GetgudSDK.Methods.Init()`. Call this method at server start in `Assets/Scripts/Multiplayer/NetworkManager.cs`.

7. When the game and match start, call `GetgudSDK.Methods.StartGame()` and `GetgudSDK.Methods.StartMatch()` with the appropriate parameters. In this example, these methods are called when the server starts in `Assets/Scripts/Multiplayer/NetworkManager.cs`.

8. To send player actions, use the appropriate GetgudSDK commands. In this example, we call `GetgudSDK.Methods.SendSpawnAction` and `GetgudSDK.Methods.SendPositionAction`. The spawn action is called [here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unity/Server/Assets/Scripts/Player.cs#L157C30-L157C45), and the position action is called [here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unity/Server/Assets/Scripts/PlayerMovement.cs#L165).

9. Send [other SDK commands](https://github.com/getgud-io/getgud-docs/blob/main/sdk-commands.md) in a similar way according to your needs. 
