# Getgud Unreal Engine Integration Tutorial

## Introduction

The `Client_Server_Getgud` project demonstrates how to integrate the GetgudSDK with Unreal Engine 5 (UE5) using a C++ based API. The project consists of a server and a client. The client sends spawn and position actions to the server, which then forwards these actions to Getgud.io's cloud.

[Video Tutorial: How To Run Unreal Engine 5 Example On Windows With Getgud SDK](https://youtu.be/H25_EuqL9c0)

[Video Tutorial: How To Run Unreal Engine 5 Example On Linux With Getgud SDK](https://youtu.be/wAIHTLxvtt0)

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

1. Clone Getgud repository: `git clone https://github.com/getgud-io/cpp-getgud-sdk-dev.git`
2. Copy `cpp-getgud-sdk-dev/examples/unreal-engine-5/Client_Server_Getgud/*` to your project folder.
3. After copy operation, you should have `Plugins` and `ThirdParty` folders. The content of the `Source` folder will be rewritten after copy.
4. Copy `cpp-getgud-sdk-dev/include` folder into `Client_Server_Getgud/ThirdParty/GetgudSDK/include` folder.
5. Copy `cpp-getgud-sdk-dev/include` and `cpp-getgud-sdk-dev/src` folders into `Client_Server_Getgud/Plugins/GetgudSDK/Source/GetgudSDKModule/ThirdParty` folder.
6. Delete the following files as they will prevent UE5 from building the code:
   - `Client_Server_Getgud/ThirdParty/GetgudSDK/include/include/GetgudSDK_C.h`
   - `Client_Server_Getgud/Plugins/GetgudSDK/Source/GetgudSDKModule/ThirdParty/include/GetgudSDK_C.h`
   - `Client_Server_Getgud/Plugins/GetgudSDK/Source/GetgudSDKModule/ThirdParty/src/GetgudSDK_C.cpp`

### 3. Build the Project

#### On Windows:

1. Open the project .sln file `Client_Server_Getgud.sln` in Visual Studio.
2. Make sure you set up `title_id` and `private_key` variables [here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unreal-engine-5/Client_Server_Getgud/Source/Client_Server_GetGud/Client_Server_GetgudPlayerController.cpp#L51).
3. Build the project to ensure all dependencies and files are correctly integrated.

#### On Linux:

1. Open UE5 `Client_Server_Getgud` project.
2. Make sure you set up `title_id` and `private_key` variables [here](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/unreal-engine-5/Client_Server_Getgud/Source/Client_Server_GetGud/Client_Server_GetgudPlayerController.cpp#L51).
3. In the bottom right corner of the screen in UE5 find and click on the button representing a grid of white squares, which recompiles the C++ code.

### 4. Add Config and Log File Paths

1. Create a `config.json` file in your project directory with the following content:

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

You can also take the config file from the [root](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/config.json) of the SDK repository.

#### Set the environment variables on Windows:

For Windows, use `setx`.
```
setx GETGUD_CONFIG_PATH "/path/to/your/config.json"
setx GETGUD_LOG_FILE_PATH "/path/to/your/logs.txt"
```

You can also use the Environment Variables menu to set the env variables.
<b>[IMPORTANT]</b> You may want to reload your Windows instance so that env variables will be seen in your system!


#### Set the environment variables on Linux:

For linux set env variables using `export` command.
```bash
export GETGUD_CONFIG_PATH=/path/to/your/config.json
export GETGUD_LOG_FILE_PATH=/path/to/your/logs.txt
```

### 5. Integrate Position data and other events correctly

In this example project we have already integrated Init, StartGame, StartMatch, PositionActionData, SpawnActionData and other events. For full list of GetgudSDK commands please visit [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/sdk-commands.md)


### 6. Run C++ Server

#### On Windows:
1. Open Visual Studio Community with `Client_Server_Getgud` project.
2. Navigate to `Project -> Properties -> Debugging`.
3. Update the `Command Arguments` field with the following:
   ```
   "$(SolutionDir)Client_Server_Getgud.uproject" -log -server -NoGraphics -port=7777
   ```
4. Click the `Local Windows Debugger` button to run the project.

#### On Linux:
1. Open the terminal window
2.  Make sure you set up `GETGUD_CONFIG_PATH` and `GETGUD_LOG_FILE_PATH` in <b>THIS</b> terminal window
3.  Run the following command from the terminal
   ```
   <PATH_TO_UNREAL_EDITOR>/UnrealEditor <PATH_TO_CLIENT_SERVER_PROJECT>/Client_Server_Getgud.uproject -log -server -NoGraphics -port=7777
   ```

### 7. Adjust Settings in UE5

1. In Edit -> Project Settings -> Maps & Modes: Set **Game Instance Class** to `GetgudInstance`
2. In Edit -> Project Settings -> Maps & Modes: Set **Server Default Map** to `FirstPersonMap`

### 8. Start UE5 Project

### 9. Connect to the Server

1. In UE5, open the console command (`~` key).
2. Connect to the server using the command:
   ```
   open 127.0.0.1:7777
   ```

### 10. Test the Integration

1. Observe the `logs.txt` to ensure that the client and server are communicating correctly.
2. Verify that actions are being sent to Getgud.io's cloud using the dashboard. [This tutorial](https://github.com/getgud-io/getgud-docs/blob/main/2-Platform/get-started-with-dashboard.md) can help you get started with Getgud.io Dashboard.

## Conclusion

Following this guide, you should have successfully integrated the GetgudSDK into the UE5 example project.
