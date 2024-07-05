# Getgud Unity Integration Tutorial

## Introduction

In this tutorial we will use [SimpleFPS project](https://github.com/RiptideNetworking/SampleFPS) as the base.

You can see full video overview of the project[here](https://www.youtube.com/watch?v=6kWNZOFcFQw&t=664s&ab_channel=TomWeiland)

In SimpleFPS we create Client and Server Unity projects, we will integrate Getgud SDK into the Server Unity project.

Before you start integration process make sure you have built Getgud SDK following [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-unity)

## Integration

1. Create "Client" and "Server" Unity project in the Unity Hub. Use `Universal 3D Sample` template.

2. Once your projects are created delete `Assets`, `Packages` and `ProjectSettings` folders in each project. Replace them with folders from [Getgud Unity example](https://github.com/getgud-io/cpp-getgud-sdk-dev/tree/main/examples/unity).

3. Put your Getgud SDK build file into `Server/Assets/Plugins/GetgudSDK/` folder.

4. Create Getgud `config.json` file in `Server/` folder. Paste our default config into `config.json`:

```
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