# Getgud Unreal Engine Integration Tutorial

## Introduction

The `Client_Server_Getgud` project demonstrates how to integrate the GetgudSDK with Unreal Engine 5 (UE5) using a C++ based API. The project consists of two parts: a server and a client. The client sends spawn and position actions to the server, which then forwards these actions to Getgud.io's cloud.

## Prerequisites

- Unreal Engine installed
- Visual Studio installed
- GetgudSDK (Ensure you have the required credentials: `titleId` and `privateKey`)

## Project Setup

### 1. Create a New UE Project

1. Open Unreal Engine.
2. Create a new FirstPerson project with C++ and Starter Content enabled.
3. Name the project `Client_Server_Getgud`. It is important to keep the project name **as is**!

### 2. Integrate GetgudSDK Files

1. Clone Getgud repository `git clone https://github.com/getgud-io/cpp-getgud-sdk-dev.git`
2. Copy `cpp-getgud-sdk-dev/examples/unreal-engine-5/Client_Server_Getgud/*` to your project folder.
3. After copy operation you should have `Plugins`, `ThirdParty` folders, content of `Source` folder will be rewritten after copy.
4. Copy `cpp-getgud-sdk-dev/include` folder into `Client_Server_Getgud/ThirdParty/GetgudSDK/include` folder.
5. Copy `cpp-getgud-sdk-dev/include` and `cpp-getgud-sdk-dev/src` folders into `Client_Server_Getgud/Plugins/GetgudSDK/Source/GetgudSDKModule/ThirdParty` folder.
6. Delete the following files as they will prevent UE5 from building the code.
   - `Client_Server_Getgud/ThirdParty/GetgudSDK/include/include/GetgudSDK_C.h`
   - `Client_Server_Getgud/Plugins/GetgudSDK/Source/GetgudSDKModule/ThirdParty/include/GetgudSDK_C.h`
   - `Client_Server_Getgud/Plugins/GetgudSDK/Source/GetgudSDKModule/ThirdParty/src/GetgudSDK_C.cpp`


### 3. Build the Project

1. Open the project sln file `Client_Server_Getgud.sln` in Visual Studio.
2. Make sure you set up `title_id` and `private_key` variables [here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unreal-engine-5/Client_Server_Getgud/Source/Client_Server_GetGud/Client_Server_GetgudPlayerController.cpp#L51)
3. Build the project to ensure all dependencies and files are correctly integrated.

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
2. Open UE5 and load the `Client_Server_Getgud` project.
3. Run the project.

### 7. Connect to the Server

1. In the UE5, open the console command (`~` key).
2. Connect to the server using the command:
   ```
   open 127.0.0.1:7777
   ```

### 8. Test the Integration

1. Observe the logs to ensure that the client and server are communicating correctly.
2. Verify that actions are being sent to and from the server, and ultimately to Getgud.io's cloud.

## Conclusion

Following this guide, you should have successfully integrated the GetgudSDK into your UE project. By setting up the server-client architecture and implementing the necessary RPC functions, you can efficiently handle and test actions like movement and spawning within the UE environment.
