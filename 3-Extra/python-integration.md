# Getgud.io Python SDK

## What Can You Do With Getgud's SDK:

Getgud's Python SDK allows you to integrate your game with the Getgud platform. Once integrated, you will be able to:

- Stream live game data to Getgud's cloud, including in-match actions, in-match reports, and in-match chat messages.
- Send Reports about historical matches to Getgud.
- Send (and update) player information to Getgud.

## Getting Started

Set the environment variables `GETGUD_TITLE_ID` and `GETGUD_PRIVATE_KEY` with the Title Id and Private Key you received from Getgud.io, or use them as arguments for StartGame. 

First, import the SDK:

```python
from getgudsdk_wrapper import GetgudSDK
import time
```

Initialize the SDK:

```python
sdk = GetgudSDK()
```

Start a Game:

```python
game_guid = sdk.start_game(
    title_id=28,
    private_key="your_private_key",
    server_guid="cs2-server-1",  # max 36 chars
    game_mode="deathmatch",  # max 36 chars
    server_location="203.0.113.42"  # max 36 chars
)
```

Once the Game starts, you'll receive the Game's guid, now you can start a Match:

```python
match_guid = sdk.start_match(
    game_guid=game_guid, 
    match_mode="Knives-only",  # max 36 chars
    map_name="de-dust"  # max 36 chars
)
```

Once you create a match, you can send Actions to it.
Let's create a spawn action and send it to the match:

```python
action_time_epoch = int(time.time() * 1000)
sdk.send_spawn_action(
    match_guid=match_guid,
    action_time_epoch=action_time_epoch,
    player_guid="player-10",
    character_guid="character-1",
    team_guid="team-1",
    initial_health=100.0,
    position=(1.0, 2.0, 3.0),
    rotation=(10.0, 20.0, 0.0)
)
```

End a game (All Game's Matches will close as well):

```python
result = sdk.mark_end_game(game_guid)
```

Flush and wait for all queued actions to be sent (optional, before shutdown):

```python
flush_result = sdk.flush()
```

Close and Dispose of the SDK:

```python
sdk.dispose()
```

## SDK Methods

### __init__()

Initializes the SDK. You can call this method in three ways:

```python
sdk = GetgudSDK()
sdk.__init_conf_path__(config_file_path)
sdk.__init_conf__(config_file, passAsContent)
```

- The first version uses default configuration.
- The second version allows you to specify a path to a configuration file.
- The third version allows you to pass the configuration content directly (if `passAsContent` is True) or as a file path (if `passAsContent` is False).

### start_game(title_id, private_key, server_guid, game_mode, server_location)

Starts a new game and returns the `game_guid`, a unique identifier for the game.

Parameters:
- `title_id` (int) - The title ID provided by Getgud.io.
- `private_key` (str) - The private key associated with the title ID.
- `server_guid` (str) - A unique name for your game server. Max 36 chars.
- `game_mode` (str) - The mode of the game. Max 36 chars.
- `server_location` (str) - The location of the server. Max 36 chars.

Returns:
- `game_guid` (str) - The unique identifier for the game.

### start_match(game_guid, match_mode, map_name)

Starts a new match within a game and returns the `match_guid`, a unique identifier for the match.

Parameters:
- `game_guid` (str) - The unique identifier for the game.
- `match_mode` (str) - The mode of the match. Max 36 chars.
- `map_name` (str) - The name of the map for the match. Max 36 chars.

Returns:
- `match_guid` (str) - The unique identifier for the match.

### mark_end_game(game_guid)

Marks a game as ended. This will close the game on the Getgud platform.

Parameters:
- `game_guid` (str) - The unique identifier for the game.

Returns:
- `result` (int) - The result of the operation (1 for success, 0 for failure).

Example:
```python
result = sdk.mark_end_game(game_guid)
```

### flush()

Waits until all queued actions are sent to the server before returning. Use this when you need to ensure all data has been transmitted before shutting down.

Returns:
- `result` (int) - 1 if all queued actions were successfully sent, 0 if the timeout was reached.

The timeout is configured via `markEndGameBlockingTimeoutMilliseconds` in config (default: 10 seconds).

Example:
```python
result = sdk.flush()
```

### send_in_match_report(match_guid, reporter_name, reporter_type, reporter_sub_type, suspected_player_guid, tb_type, tb_time_epoch, suggested_toxicity_score, reported_time_epoch)

Sends a report for a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match. Max 36 chars.
- `reporter_name` (str) - The name of the entity that created the report. Max 36 chars.
- `reporter_type` (int) - The type of the entity that created the report.
- `reporter_sub_type` (int) - The subtype of the entity that created the report.
- `suspected_player_guid` (str) - The player Id of the suspected player. Max 36 chars.
- `tb_type` (int) - Id of the toxic behavior type, for example, Aimbot.
- `tb_time_epoch` (int) - Epoch time of when the toxic behavior event occurred.
- `suggested_toxicity_score` (int) - 0-100 toxicity score, i.e., how much you suspect the player.
- `reported_time_epoch` (int) - Epoch time of when the report was created.

Returns:
- `result` (int) - The result of the operation (0 for success).

### send_chat_message(match_guid, message_time_epoch, player_guid, message)

Sends a chat message to a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match. Max 36 chars.
- `message_time_epoch` (int) - Epoch time of when the message was sent.
- `player_guid` (str) - The unique identifier of the player sending the message. Max 36 chars.
- `message` (str) - The content of the chat message. Max 256 chars.

Returns:
- `result` (int) - The result of the operation (0 for success).

### send_attack_action(match_guid, action_time_epoch, player_guid, weapon_guid)

Sends an attack action to a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match. Max 36 chars.
- `action_time_epoch` (int) - Epoch time of when the action occurred.
- `player_guid` (str) - The unique identifier of the player performing the action. Max 36 chars.
- `weapon_guid` (str) - The unique identifier of the weapon used. Max 36 chars.

Returns:
- `result` (int) - The result of the operation (0 for success).

### send_damage_action(match_guid, action_time_epoch, player_guid, victim_player_guid, damage_done, weapon_guid)

Sends a damage action to a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match. Max 36 chars.
- `action_time_epoch` (int) - Epoch time of when the action occurred.
- `player_guid` (str) - The unique identifier of the player performing the action. Max 36 chars.
- `victim_player_guid` (str) - The unique identifier of the player receiving the damage. Max 36 chars.
- `damage_done` (float) - The amount of damage dealt.
- `weapon_guid` (str) - The unique identifier of the weapon used. Max 36 chars.

Returns:
- `result` (int) - The result of the operation (0 for success).

### send_heal_action(match_guid, action_time_epoch, player_guid, health_gained)

Sends a heal action to a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match. Max 36 chars.
- `action_time_epoch` (int) - Epoch time of when the action occurred.
- `player_guid` (str) - The unique identifier of the player being healed. Max 36 chars.
- `health_gained` (float) - The amount of health gained.

Returns:
- `result` (int) - The result of the operation (0 for success).

### send_spawn_action(match_guid, action_time_epoch, player_guid, character_guid, team_guid, initial_health, position, rotation)

Sends a spawn action to a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match. Max 36 chars.
- `action_time_epoch` (int) - Epoch time of when the action occurred.
- `player_guid` (str) - The unique identifier of the player spawning. Max 36 chars.
- `character_guid` (str) - The unique identifier of the character being spawned. Max 36 chars.
- `team_guid` (str) - The unique identifier of the team. Max 36 chars.
- `initial_health` (float) - The initial health of the spawned character.
- `position` (tuple) - The spawn position as (X, Y, Z).
- `rotation` (tuple) - The spawn rotation as (Yaw, Pitch, Roll).

Returns:
- `result` (int) - The result of the operation (0 for success).

### send_death_action(match_guid, action_time_epoch, player_guid, attacker_guid)

Sends a death action to a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match. Max 36 chars.
- `action_time_epoch` (int) - Epoch time of when the action occurred.
- `player_guid` (str) - The unique identifier of the player who died. Max 36 chars.
- `attacker_guid` (str) - The unique identifier of the attacker. Max 36 chars.

Returns:
- `result` (int) - The result of the operation (0 for success).

### send_position_action(match_guid, action_time_epoch, player_guid, position, rotation)

Sends a position action to a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match. Max 36 chars.
- `action_time_epoch` (int) - Epoch time of when the action occurred.
- `player_guid` (str) - The unique identifier of the player. Max 36 chars.
- `position` (tuple) - The new position as (X, Y, Z).
- `rotation` (tuple) - The new rotation as (Yaw, Pitch, Roll).

Returns:
- `result` (int) - The result of the operation (0 for success).

### send_report(title_id, private_key, match_guid, reporter_name, reporter_type, reporter_sub_type, suspected_player_guid, tb_type, tb_time_epoch, suggested_toxicity_score, reported_time_epoch)

Sends a report for a historical match (a match which is not live and has ended).

Parameters:
- `title_id` (int) - The title ID provided by Getgud.io.
- `private_key` (str) - The private key associated with the title ID.
- Other parameters are the same as `send_in_match_report`.

Returns:
- `result` (int) - The result of the operation (0 for success).

### send_affect_action(match_guid, action_time_epoch, player_guid, affect_guid, affect_state)

Sends an affect action to a match.

Parameters:
- `match_guid` (str) - The unique identifier for the match. Max 36 chars.
- `action_time_epoch` (int) - Epoch time of when the action occurred.
- `player_guid` (str) - The unique identifier of the player. Max 36 chars.
- `affect_guid` (str) - The unique identifier of the affect. Max 36 chars.
- `affect_state` (GetgudSDK.AffectState or str) - The state of the affect (Attach, Activate, Deactivate, Detach).

Returns:
- `result` (int) - The result of the operation (0 for success).

### update_player(title_id, private_key, player_guid, player_nickname, player_email, player_rank, player_join_date_epoch, player_suspect_score, player_reputation, player_status, player_campaign, player_notes, player_device, player_os, player_age, player_gender, player_location, transactions)

Updates player information.

Parameters:
- `title_id` (int) - The title ID provided by Getgud.io.
- `private_key` (str) - The private key associated with the title ID.
- `player_guid` (str) - The unique identifier of the player. Max 36 chars. **(Mandatory field)**
- `player_nickname` (str) - The nickname of the player. Max 36 chars.
- `player_email` (str) - The email of the player. Max 36 chars.
- `player_rank` (int) - The rank of the player.
- `player_join_date_epoch` (int) - The join date of the player in epoch time.
- `player_suspect_score` (str) - The suspect score of the player.
- `player_reputation` (str) - The reputation of the player. Max 36 chars.
- `player_status` (str) - The status of the player. Max 36 chars.
- `player_campaign` (str) - The campaign of the player. Max 128 chars.
- `player_notes` (str) - Notes about the player. Max 128 chars.
- `player_device` (str) - The device of the player. Max 8 chars.
- `player_os` (str) - The OS of the player. Max 8 chars.
- `player_age` (int) - The age of the player.
- `player_gender` (str) - The gender of the player. Max 8 chars.
- `player_location` (str) - The location of the player. Max 36 chars.
- `transactions` (list) - A list of player transactions.

Returns:
- `result` (int) - The result of the operation (0 for success).

### dispose()

Disposes of the SDK when it's no longer needed.

## Configuration and Logging

The default Config file is loaded during initialization. You can specify a custom configuration file path or pass the configuration content directly using the `__init_conf_path__` or `__init_conf__` methods.

Example of configuration file `config.json`:

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
  "markEndGameBlockingTimeoutMilliseconds": 10000,
  "logLevel": "FULL"
}
```

### Logging

The SDK logs its operations based on the `logLevel` setting in the configuration file. To control logging, use the following configuration parameters:

- `logToFile`: Set to `true` to enable logging to file, `false` to disable.
- `logFileSizeInBytes`: Maximum size of the log file before it's rotated (default: 10 MB).
- `circularLogFile`: If `true`, the log file will be overwritten from the beginning when it reaches the maximum size. If `false`, a new log file will be created.
- `logLevel`: Controls the verbosity of logging. Options are "FULL", "DEBUG", "INFO", "WARNING", "ERROR", and "FATAL".

To change the location of the log file, you can set the `GETGUD_LOG_FILE_PATH` environment variable to the desired file path.

## Examples

Here's a more comprehensive example of using the Getgud Python SDK:

```python
from getgudsdk_wrapper import GetgudSDK
import time

# Initialize the SDK
sdk = GetgudSDK()

# Start a game
game_guid = sdk.start_game(
    title_id=28,
    private_key="your_private_key",
    server_guid="cs2-server-1",
    game_mode="deathmatch",
    server_location="203.0.113.42"
)

# Start a match
match_guid = sdk.start_match(
    game_guid=game_guid,
    match_mode="Knives-only",
    map_name="de-dust"
)

# Send a spawn action
action_time_epoch = int(time.time() * 1000)
sdk.send_spawn_action(
    match_guid=match_guid,
    action_time_epoch=action_time_epoch,
    player_guid="player-10",
    character_guid="character-1",
    team_guid="team-1",
    initial_health=100.0,
    position=(1.0, 2.0, 3.0),
    rotation=(10.0, 20.0, 0.0)
)

# Send a position action
sdk.send_position_action(
    match_guid=match_guid,
    action_time_epoch=int(time.time() * 1000),
    player_guid="player-10",
    position=(2.0, 3.0, 4.0),
    rotation=(15.0, 25.0, 5.0)
)

# Send an attack action
sdk.send_attack_action(
    match_guid=match_guid,
    action_time_epoch=int(time.time() * 1000),
    player_guid="player-10",
    weapon_guid="weapon-1"
)

# Send a damage action
sdk.send_damage_action(
    match_guid=match_guid,
    action_time_epoch=int(time.time() * 1000),
    player_guid="player-10",
    victim_player_guid="player-11",
    damage_done=25.0,
    weapon_guid="weapon-1"
)

# Send a heal action
sdk.send_heal_action(
    match_guid=match_guid,
    action_time_epoch=int(time.time() * 1000),
    player_guid="player-10",
    health_gained=50.0
)

# Send a death action
sdk.send_death_action(
    match_guid=match_guid,
    action_time_epoch=int(time.time() * 1000),
    player_guid="player-11",
    attacker_guid="player-10"
)

# Send a chat message
sdk.send_chat_message(
    match_guid=match_guid,
    message_time_epoch=int(time.time() * 1000),
    player_guid="player-10",
    message="Hello, world!"
)

# Send an in-match report
sdk.send_in_match_report(
    match_guid=match_guid,
    reporter_name="player-10",
    reporter_type=1,
    reporter_sub_type=1,
    suspected_player_guid="player-11",
    tb_type=1,
    tb_time_epoch=int(time.time() * 1000),
    suggested_toxicity_score=75,
    reported_time_epoch=int(time.time() * 1000)
)

# Update player information
sdk.update_player(
    title_id=28,
    private_key="your_private_key",
    player_guid="player-10",
    player_nickname="John Doe",
    player_email="john@example.com",
    player_rank=10,
    player_join_date_epoch=int(time.time() * 1000),
    player_suspect_score="50",
    player_reputation="Good",
    player_status="Active",
    player_campaign="Summer2023",
    player_notes="Skilled player",
    player_device="PC",
    player_os="Windows",
    player_age=25,
    player_gender="Male",
    player_location="New York",
    transactions=[
        {
            "TransactionGuid": "trans-001",
            "TransactionName": "In-Game Purchase",
            "TransactionDateEpoch": int(time.time() * 1000),
            "TransactionValueUSD": 9.99
        }
    ]
)

# End the game
sdk.mark_end_game(game_guid)

# Optional: Flush to ensure all data is sent before shutdown
sdk.flush()

# Dispose of the SDK
sdk.dispose()
```

This example demonstrates how to use various methods of the Getgud Python SDK, including starting a game and match, sending different types of actions, sending chat messages and reports, updating player information, and ending the game.

Remember to replace "your_private_key" with your actual private key provided by Getgud.io, and adjust other parameters as needed for your specific game implementation.

For more detailed examples and use cases, refer to the [examples](https://github.com/getgud-io/cpp-getgud-sdk-dev/tree/main/examples) directory in the Getgud SDK repository.
