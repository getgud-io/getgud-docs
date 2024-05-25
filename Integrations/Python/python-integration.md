# GetGud Python SDK

## What Can You Do With GetGud's SDK

Getgud Python SDK allows you to integrate your game with the GetGud platform. Once integrated, you will be able to:
- Stream live game data to GetGud's cloud, including in-match actions, in-match reports, and in-match chat messages.
- Send Reports about historical matches to GetGud.
- Send (and update) player information to GetGud.

## Build requirements:
The deployment folder contains everything needed to build and link the SDK into a Python project.
However, for the Python version of the SDK, dependencies are hidden from the user:
### Linux:
Dependencies:
- libcurl development kit.
- zlib development kit.
- openssl development kit.
- libssl development kit.
- libcrypto development kit.
- g++ and gcc packages.
- pthread library.

### Windows:
- libcurl library package.
- zlib library package.

Depending on the local machine's environment, some additional library linking might be required. 
A test project with a complete setup is included in the root folder.

## Prerequisites

For more context please check [CPP SDK Docs](https://github.com/getgud-io/cpp-getgud-sdk)
To start, we need to understand the basic structure GetGud's SDK uses for an FPS: 

**Titles->1->N->Games->1->N->Matches->1->N->Actions**

* The top container in Getgud's SDK is `Title`, which represents a literal game’s title, you as a client can have many titles, for example, a `Title` named CS:GO represents the CS:GO video game.

  ```
  An example of a Title: CS:GO 
  ```

* Next up is `Game`, a `Game` is a container of matches that belong to the same `Title` from the same server session, where mostly the same players in the same teams, play one or more `Matches` together. You as a client can identify every game with a unique `gameGuid` that is provided to you once the `Game` starts. 

  ```
  An example of a Game is a CS:GO game which has 30 macthes (AKA rounds) inside it.
  ```

* `Match` represents the actual play time that is streamed for analysis.
A `Match` contains the actions that occurred during the match's timespan.
Like `Game`, `Match` also has a GUID which will be provided to you once you start a new match.

  ```
  An example of a Match is a single CS:GO round inside the game.
  ```

* `Action` represents an in-match activity that is associated with a player. We collect six different action types which are common to all first person shooter gamnes:
1. `Spwan` - Whenever a player appears or reappears in-match, on the map.
2. `Death` - A death of a player.
3. `Position` - player position change (including looking direction).
4. `Attack` - Whenever a player initiates any action that might cause damage, now or in the future. Examples: shooting, throwning a granade, planting a bomb, swinging a sword, punching, firing a photon torpedo, etc.
5. `Damage` - Whenever a player recieves any damage, from players or the environment.
6. `Heal` - Whenever a player is healed.


## Getting Started

First, you need to build the SDK for your system. Follow these steps:

1. Install Miniconda by executing the following commands:

   ```bash
   wget https://repo.anaconda.com/miniconda/Miniconda3-py39_23.3.1-0-Linux-x86_64.sh
   bash Miniconda3-py39_23.3.1-0-Linux-x86_64.sh
   source miniconda3/bin/activate
   conda create -n getgudsdk python=3.11 anaconda
   conda activate getgudsdk
   pip install -r requirements.txt
   ```

   For Windows, you can download Miniconda from [here](https://docs.conda.io/en/latest/miniconda.html).

2. Download the latest Windows or Linux library build files from GetGud's S3 bucket.

3. Run the following command in your terminal to build the library:

   ```bash
   ln -s libGetGudSdk.so.0 libGetGudSdk.so
   invoke build-sdk
   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/your/python-getgud-sdk
   ```

   On Windows, add the path to the SDK .pyd file to PYTHONPATH.

4. Set the `LOG_FILE_PATH` and `CONFIG_PATH` environment variables to use the SDK.

Now let's get started with the SDK!

To use the GetGud Python SDK, you need to import the `GetGudSdk` class and other necessary modules:

```python
from getgudsdk_wrapper import GetGudSdk
import time
import random
```

Initialize the SDK:

```python
sdk = GetGudSdk()
```

Start a Game:

```python
game_guid = sdk.start_game(1, "private_key", "server_guid", "game_mode")
```

Once the Game starts, you can start a Match:

```python
match_guid = sdk.start_match(game_guid, "match_mode", "map_name")
```

Now you can push Actions, Chat Data, and Reports to the Match. Let's push a Spawn Action to this match:

```python
action_time_epoch = int(time.time() * 1000)
player_guid = "player_1"
character_guid = "ttr"
position = (1, 2, 3)
rotation = (10, 20, 30)
team_id = 1
initial_health = 100

sdk.send_spawn_action(match_guid, action_time_epoch, player_guid, character_guid, team_id, initial_health, position, rotation)
```

Create one more match and push a report:

```python
match_guid = sdk.start_match(game_guid, "deathmatch", "de-mirage")

reporter_name = "player1"
reporter_type = 0
reporter_sub_type = 0
suspected_player_guid = "player1"
tb_type = 0
tb_sub_type = 0
tb_time_epoch = int(time.time() * 1000)
suggested_toxicity_score = 100
reported_time_epoch = int(time.time() * 1000)

sdk.send_in_match_report(match_guid, reporter_name, reporter_type, reporter_sub_type, 
                        suspected_player_guid, tb_type, tb_sub_type, 
                        tb_time_epoch, suggested_toxicity_score, reported_time_epoch)
```

Stop the Game:

```python
sdk.mark_end_game(game_guid)
```

Finally, dispose of the SDK when you no longer need it:

```python
sdk.dispose()
```

## SDK Methods

### start_game

Starts a new game and returns the `gameGuid`, a unique identifier for the game.

Parameters:
- `title_id` - The title ID provided by GetGud.io.
- `private_key` (str) - The private key associated with the title ID.
- `server_name` (str) - A unique name for your game server.
- `game_mode` (str) - The mode of the game.

Returns:
- `game_guid` (str) - The unique identifier for the game.

### start_match

Starts a new match within a game and returns the `matchGuid`, a unique identifier for the match.

Parameters:
- `game_guid` (str) - The unique identifier for the game.
- `match_mode` (str) - The mode of the match.
- `map_name` (str) - The name of the map for the match.

Returns:
- `match_guid` (str) - The unique identifier for the match.

### send_action

Sends an action to a match. Actions can include spawn, position, attack, damage, heal, and death.

Parameters:
- `match_guid` (str) - The unique identifier for the match.
- `action` (dict) - The action data.

### mark_end_game

Marks a game as ended. This will close the game on the GetGud platform.

Parameters:
- `game_guid` (str) - The unique identifier for the game.

Returns:
- `game_ended` (bool) - Whether the game was successfully closed or not.

### send_chat_message

Sends a chat message to a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match.
- `message_data` (dict) - The chat message data.

### send_in_match_report

Sends a report for a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match.
- `reported_time_epoch` (int) - epoch time of when the report was created
- `reporter_name` (str) - the name of the entity that created the report
- `reporter_type` (int) - the type of the entity that created the report
- `reporter_sub_type` (int) -   the subtype of the entity that created the report
- `suggested_toxicity_score` (int) - 0-100 toxicity score, ie: how much do you suspect the player
- `suspected_player_id` (int) - the player Id of the suspected player
- `tb_type` (int) - id of the toxic behavior type, for example, Aimbot
- `tb_time_epoch` (int) - epoch time of when the toxic behavior event occured

### send_report

Send report which are outside of the live match.

Parameters:
- `match_guid` (str) - The unique identifier for the match.
- `reported_time_epoch` (int) - epoch time of when the report was created
- `reporter_name` (str) - the name of the entity that created the report
- `reporter_type` (int) - the type of the entity that created the report
- `reporter_sub_type` (int) -   the subtype of the entity that created the report
- `suggested_toxicity_score` (int) - 0-100 toxicity score, ie: how much do you suspect the player
- `suspected_player_id` (int) - the player Id of the suspected player
- `tb_type` (int) - id of the toxic behavior type, for example, Aimbot
- `tb_time_epoch` (int) - epoch time of when the toxic behavior event occured

### update_player

Update player info outside of the live match.

Parameters:
- `title_id` (int) - The title ID provided by GetGud.io (optional if using environment variables).
- `private_key` (str) - The private key associated with the title ID (optional if using environment variables).
- `player_guid` (str) - Your player Id - String, 36 chars max.
- `player_nickname` (str) - Nickname of the player.
- `player_email` (str) - Email of the player.
- `player_rank` (int) - Rank of the player.
- `player_join_date_epoch` (int) - Date when the player joined.

Returns:
- `players_updated` (bool) - Whether the player information was successfully updated or not.

### dispose

Disposes of the SDK when no longer needed.



## Configuration

The Config JSON file is loaded during initialization using the `CONFIG_PATH` environment variable.
Example of configuration file `config.json`:

```json
{
  "throttleCheckUrl": "https://www.getgud.io/api/game_stream/throttle_match_check",
  "streamGameURL": "https://www.getgud.io/api/game_stream/send_game_packet",
  "updatePlayersURL": "https://www.getgud.io/api/player_data/update_players",
  "sendReportsURL": "https://www.getgud.io/api/report_data/send_reports",
  "logToFile": true,
  "logFileSizeInBytes": 2000000,
  "circularLogFile": true,
  "reportsMaxBufferSizeInBytes": 100000,
  "maxReportsToSendAtOnce": 100,
  "maxChatMessagesToSendAtOnce": 100,
  "playersMaxBufferSizeInBytes": 100000,
  "maxPlayerUpdatesToSendAtOnce": 100,
  "gameSenderSleepIntervalMilliseconds": 100,
  "apiTimeoutMilliseconds": 600,
  "apiWaitTimeMilliseconds": 100,
  "packetMaxSizeInBytes": 2000000,
  "actionsBufferMaxSizeInBytes": 10000000,
  "gameContainerMaxSizeInBytes": 50000000,
  "maxConcurrentGames": 100,
  "maxMatchesPerGame": 30,
  "minPacketSizeForSendingInBytes": 1000000,
  "packetTimeoutInMilliseconds": 100000,
  "gameCloseGraceAfterMarkEndInMilliseconds": 20000,
  "liveGameTimeoutInMilliseconds": 100000,
  "hyperModeFeatureEnabled": true,
  "hyperModeMaxThreads": 10,
  "hyperModeAtBufferPercentage": 10,
  "hyperModeUpperPercentageBound": 90,
  "hyperModeThreadCreationStaggerMilliseconds": 100,
  "logLevel": "warn"
}
```

Please note that the SDK will not start if `CONFIG_PATH` is not set. Adjust the values in the configuration file according to your application's requirements.

## Logging

The SDK logs its actions based on the `logLevel` setting in the configuration file. Logging will only work if the `LOG_FILE_PATH` environment variable is set.

To control logging, use the following configuration parameters:

- `logToFile`: true/false - Whether to log to a file or not.
- `logFileSizeInBytes`: Maximum log file size in bytes (0-100000000).
- `circularLogFile`: true/false - Whether to remove the first lines of the log file when it exceeds the size limit.

Please make sure to set the `LOG_FILE_PATH` and `CONFIG_PATH` environment variables correctly to use logging.
