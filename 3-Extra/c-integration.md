# Getgud.io C SDK

## What Can You Do With Getgud's SDK:

Getgud's C SDK allows you to integrate your game with the GetGud platform. Once integrated, you will be able to:

- Stream live game data to GetGud's cloud, including in-match actions, in-match reports, and in-match chat messages.
- Send Reports about historical matches to Getgud.
- Send (and update) player information to Getgud.

## Getting Started

Set the environment variables `GETGUD_TITLE_ID` and `GETGUD_PRIVATE_KEY` with the Title Id and Private Key you received from Getgud.io, or use them as arguments for StartGame. 

Include the following header file:

```c
#include <GetgudSDK_C.h>
```

Initialize the SDK:

```c
int result = init();
```

Start a Game:

```c
struct StartGameInfo gameInfo;
gameInfo.serverGuid = "cs2-server-1"; // max 36 chars
gameInfo.serverGuidSize = strlen(gameInfo.serverGuid);
gameInfo.gameMode = "deathmatch"; // max 36 chars
gameInfo.gameModeSize = strlen(gameInfo.gameMode);
gameInfo.serverLocation = "203.0.113.42"; // max 36 chars
gameInfo.serverLocationSize = strlen(gameInfo.serverLocation);

char gameGuid[37]; // 36 chars + null terminator
int gameGuidSize = StartGame(gameInfo, gameGuid);
```

Once the Game starts, you'll receive the Game's guid, now you can start a Match:

```c
struct StartMatchInfo matchInfo;
matchInfo.gameGuid = gameGuid;
matchInfo.gameGuidSize = gameGuidSize;
matchInfo.matchMode = "Knives-only"; // max 36 chars
matchInfo.matchModeSize = strlen(matchInfo.matchMode);
matchInfo.mapName = "de-dust"; // max 36 chars
matchInfo.mapNameSize = strlen(matchInfo.mapName);

char matchGuid[37]; // 36 chars + null terminator
int matchGuidSize = StartMatch(matchInfo, matchGuid);
```

Once you create a match, you can send Actions to it.
Let's send a spawn action:

```c
struct BaseActionData baseData;
baseData.actionTimeEpoch = getCurrentEpochTime();
baseData.matchGuid = matchGuid;
baseData.matchGuidSize = matchGuidSize;
baseData.playerGuid = "player-10";
baseData.playerGuidSize = strlen(baseData.playerGuid);

const char* characterGuid = "character-1";
const char* teamGuid = "team-1";
float initialHealth = 100.0f;
struct PositionF position = {1.0f, 2.0f, 3.0f};
struct RotationF rotation = {10.0f, 20.0f, 0.0f};

int result = SendSpawnAction(baseData, characterGuid, strlen(characterGuid), 
                             teamGuid, strlen(teamGuid), initialHealth, 
                             position, rotation);
```

End a game (All Game's Matches will close as well):

```c
int gameEnded = MarkEndGame(gameGuid, gameGuidSize);
```

Flush and wait for all queued actions to be sent (optional, before shutdown):

```c
int flushResult = Flush();
```

Close and Dispose of the SDK:

```c
dispose();
```

## SDK Methods

### init()

Initializes the SDK. You can call this method in three ways:

```c
int init();
int init_conf_path(const char* configFilePath);
int init_conf(const char* configFile, int passAsContent);
```

- The first version uses default configuration.
- The second version allows you to specify a path to a configuration file.
- The third version allows you to pass the configuration content directly (if `passAsContent` is true) or as a file path (if `passAsContent` is false).

### StartGame(struct StartGameInfo gameInfo, char* gameGuidOut)

To start a new game, call `StartGame()`, this will use the environment variables `GETGUD_TITLE_ID` and `GETGUD_PRIVATE_KEY`.
`StartGame` returns the size of the game guid, which you will use later to start new Matches inside the Game as well as to end the Game when it is over.

```c
int gameGuidSize = StartGame(gameInfo, gameGuidOut);
```
* `gameInfo` - A structure containing game information:
  * `serverGuid` - a unique name of your game server - String, Alphanumeric, max 36 chars.
  * `gameMode` - the mode of the game (each mode has a separate AI learning processes) - String, Alphanumeric, max 36 chars.
  * `serverLocation` - a name/ip of your game server's location - String, Alphanumeric, max 36 chars.
* `gameGuidOut` - A pre-allocated char array to store the output game guid (should be at least 37 bytes).

### StartMatch(struct StartMatchInfo matchInfo, char* matchGuidOut)

Once you've started a live Game, you can now attach Matches to that Game.
When a live Match starts it returns a `matchGuid`, which is used to add Actions, Chat messages, and Reports to that specific match.

```c
int matchGuidSize = StartMatch(matchInfo, matchGuidOut);
```
* `matchInfo` - A structure containing match information:
  * `gameGuid` - The Game guid you received when starting a new Game
  * `matchMode` - the mode of the match (each mode has an additional AI learning processes) - String, Alphanumeric, max 36 chars.
  * `mapName` - the unique name of the map - String, Alphanumeric, max 36 chars.
* `matchGuidOut` - A pre-allocated char array to store the output match guid (should be at least 37 bytes).

### MarkEndGame(const char* gameGuid, int guidSize)

Ends a live Game and its associated matches.
When the live Game ends, you should mark it as finished in order to close it on Getgud's side.
Once the game is marked as ended, it will not accept new live data.

```c
int gameEnded = MarkEndGame(gameGuid, guidSize);
```
* `gameGuid` - The Game guid you received when starting a new Game
* `guidSize` - The size of the game guid

`MarkEndGame` returns 1 if the Game was successfully closed, 0 otherwise.

### Flush()

Waits until all queued actions are sent to the server before returning. Use this when you need to ensure all data has been transmitted before shutting down.

```c
int success = Flush();
```

`Flush` returns `1` if all queued actions were successfully sent, or `0` if the timeout was reached. The timeout is configured via `flushTimeoutMilliseconds` in config (default: 10 seconds).

## Sending Actions 

When a live Match starts, you can add Actions, Chat Data, and Reports to it. 
There are 7 Action types you can add to the Match, all derived from base action which holds the actions base properties.

### Base Action

`BaseActionData` is the base structure that all actions use and holds common information for all actions.

```c
struct BaseActionData {
    long long actionTimeEpoch;
    const char* matchGuid;
    int matchGuidSize;
    const char* playerGuid;
    int playerGuidSize;
};
```
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `matchGuid` - guid of the live Match where the action happened, is given to you when `StartMatch` is called. Max 36 chars.
* `playerGuid` - your player Id (which will be called playerGuid on Getgud's side). Max 36 chars.

### Spawn Action

A Spawn action should be sent whenever a player appears or reappears in-match, on the map.
To create a Spawn Action, use the `SendSpawnAction` function.

```c
int SendSpawnAction(struct BaseActionData baseData,
    const char* characterGuid,
    int characterGuidSize,
    const char* teamGuid,
    int teamGuidSize,
    float initialHealth,
    struct PositionF position,
    struct RotationF rotation);
```
* `baseData` - See BaseActionData
* `characterGuid` - Guid of the spawned character from your game. Max 36 chars.
* `teamGuid` - The identifier of the player's team. Max 36 chars.
* `initialHealth` - The initial health of the player
* `position` - X,Y,Z coordinates of the player at the moment of action.
* `rotation` - YAW, PITCH, ROLL rotation of player view at the moment of action.

### Position Action

A Position action should be sent on player position change (including looking direction).
To create a Position Action, use the `SendPositionAction` function.

```c
int SendPositionAction(struct BaseActionData baseData,
    struct PositionF position,
    struct RotationF rotation);
```
* `baseData` - See BaseActionData
* `position` - X,Y,Z coordinates of the player at the moment of action.
* `rotation` - YAW, PITCH, ROLL rotation of player view at the moment of action.

### Attack Action

An Attack action is any attempt to create damage, now or in the future, for example, firing a shot, swinging a sword, placing a bomb, throwing a grenade, or any other action that may result in damage.
To create an Attack Action, use the `SendAttackAction` function.

```c
int SendAttackAction(struct BaseActionData baseData,
    const char* weaponGuid,
    int weaponGuidSize);
```
* `baseData` - See BaseActionData
* `weaponGuid` - A unique name of the weapon that the attack was performed with. Max 36 chars.

### Affect Action

An Affect action should be sent whenever an in-match affect of any kind is applied to the player. Examples: crouch, prone, jump, fly, use special ability, boost speed/ammo/shield/health, etc. 
To create an Affect Action, use the `SendAffectAction` function.

```c
int SendAffectAction(struct BaseActionData baseData,
    const char* affectGuid,
    int affectGuidSize,
    enum AffectState affectState);
```
* `baseData` - See BaseActionData
* `affectGuid` - A unique name of the affect. Max 36 chars.
* `affectState` - An enumeration unit that allows you to flag the state of the Affect:
  	Attach - indicate affect is attached to a player (not mandatory).
  	Detach - indicate affect is detached from a player (not mandatory).
  	Activate - indicate affect is affecting the player.
  	Deactivate - indicate affect stopped affecting the player.

### Damage Action

A Damage action should be raised when a player loses health, both PVP and PVE.
To create a Damage Action, use the `SendDamageAction` function.

```c
int SendDamageAction(struct BaseActionData baseData,
    const char* victimPlayerGuid,
    int victimPlayerGuidSize,
    float damageDone,
    const char* weaponGuid,
    int weaponGuidSize);
```
* `baseData` - See BaseActionData
* `victimPlayerGuid` - guid of the player who is the victim of the Damage action. Max 36 chars.
* `damageDone` - How much damage was given
* `weaponGuid` - A unique name of the weapon that the attack was performed with. Max 36 chars.

### Heal Action

A Heal action should be triggered whenever a player is healed, doesn't matter by whom or how.
To create a heal action, use the `SendHealAction` function.

```c
int SendHealAction(struct BaseActionData baseData, float healthGained);
```
* `baseData` - See BaseActionData
* `healthGained` - How much health the player gained.

### Death Action

A Death action should be triggered on a death of a player, either by another player, the environment or the player itself.
To create a death action to a match, use the `SendDeathAction` function.

```c
int SendDeathAction(struct BaseActionData baseData,
    const char* attackerGuid,
    int attackerGuidSize);
```
* `baseData` - See BaseActionData
* `attackerGuid` - guid of the player who killed the victim of the Damage action. Max 36 chars.

## Adding Chat Messages
To add a chat message to a live Match:

```c
struct ChatMessageInfo messageInfo;
messageInfo.messageTimeEpoch = getCurrentEpochTime();
messageInfo.matchGuid = matchGuid;
messageInfo.matchGuidSize = matchGuidSize;
messageInfo.playerGuid = "player1";
messageInfo.playerGuidSize = strlen(messageInfo.playerGuid);
messageInfo.message = "Hi from Getgud!";
messageInfo.messageSize = strlen(messageInfo.message);

int result = SendChatMessage(messageInfo);
```
* `messageTimeEpoch` - epoch time in milliseconds when the message was sent
* `matchGuid` - The guid of the match where the message was sent. Max 36 chars.
* `playerGuid` - The guid of the player who sent the message. Max 36 chars.
* `message` - The content of the chat message. Max 256 chars.

## Adding Reports

To add a report to a live Match:
Note that all of the fields are optional except `matchGuid`, `reportedTimeEpoch`, and `suspectedPlayerGuid` (a report must have a valid match, time, and player).
This is an async method which will not block the calling thread.

```c
struct ReportInfo reportInfo;
reportInfo.matchGuid = matchGuid;
reportInfo.matchGuidSize = matchGuidSize;
reportInfo.reportedTimeEpoch = getCurrentEpochTime();
reportInfo.reporterName = "player1";
reportInfo.reporterNameSize = strlen(reportInfo.reporterName);
reportInfo.reporterType = 1; // Moderator
reportInfo.reporterSubType = 1; // QA
reportInfo.suggestedToxicityScore = 100;
reportInfo.suspectedPlayerGuid = "player2";
reportInfo.suspectedPlayerGuidSize = strlen(reportInfo.suspectedPlayerGuid);
reportInfo.tbTimeEpoch = getCurrentEpochTime();
reportInfo.tbType = 1; // Wallhack

int result = SendInMatchReport(reportInfo);
```
* `matchGuid` - guid of the live Match you are sending a report to. Max 36 chars. **(Mandatory field)**
* `reportedTimeEpoch` - epoch time in milliseconds of when the report was created **(Mandatory field)**
* `reporterName` - the name of the entity that created the report. Max 36 chars. **(optional field)**
* `reporterType` - the type of the entity that created the report **(optional field)**
* `reporterSubType` - the subtype of the entity that created the report **(optional field)**
* `suggestedToxicityScore` - 0-100 toxicity score, ie: how much do you suspect the player **(optional field)**
* `suspectedPlayerGuid` - the player Id of the suspected player as appears in your system. Max 36 chars. **(Mandatory field)**
* `tbType` - id of the toxic behavior type, for example, Aimbot **(optional field)**
* `tbTimeEpoch` - epoch time of when the toxic behavior event occurred **(optional field)**

### Sending Reports On Historical Matches

To add reports to a historical Match (a match which is not live and has ended):

```c
int SendReport(int titleId, const char* privateKey, int privateKeySize, struct ReportInfo report);
```

## Sending Player Updates

To update player information, you can call `UpdatePlayer`:
All fields are optional except player Id.
This is an async method which will not block the calling thread.

```c
struct PlayerInfo playerInfo;
playerInfo.playerGuid = "549cf69d-0d55-4849-b2d1-a49a4f0a0b1e";
playerInfo.playerGuidSize = strlen(playerInfo.playerGuid);
playerInfo.playerNickname = "test";
playerInfo.playerNicknameSize = strlen(playerInfo.playerNickname);
playerInfo.playerEmail = "test@test.com";
playerInfo.playerEmailSize = strlen(playerInfo.playerEmail);
playerInfo.playerRank = 10;
playerInfo.playerJoinDateEpoch = getCurrentEpochTime();
playerInfo.playerSuspectScore = "12";
playerInfo.playerSuspectScoreSize = strlen(playerInfo.playerSuspectScore);
playerInfo.playerReputation = "Toxic";
playerInfo.playerReputationSize = strlen(playerInfo.playerReputation);
playerInfo.playerStatus = "Banned";
playerInfo.playerStatusSize = strlen(playerInfo.playerStatus);
playerInfo.PlayerCampaign = "Tiktok-2023-Feb-23-Id1332";
playerInfo.PlayerCampaignSize = strlen(playerInfo.PlayerCampaign);
playerInfo.playerDevice = "PC";
playerInfo.playerDeviceSize = strlen(playerInfo.playerDevice);
playerInfo.playerOS = "Win11";
playerInfo.playerOSSize = strlen(playerInfo.playerOS);
playerInfo.playerAge = 27;
playerInfo.playerGender = "Male";
playerInfo.playerGenderSize = strlen(playerInfo.playerGender);
playerInfo.playerLocation = "203.0.113.42";
playerInfo.playerLocationSize = strlen(playerInfo.playerLocation);
playerInfo.playerNotes = "I just banned this player from my system because of XYZ";
playerInfo.playerNotesSize = strlen(playerInfo.playerNotes);

// Add player transaction
struct PlayerTransactions transaction;
transaction.TransactionGuid = "trans-001";
transaction.TransactionGuidSize = strlen(transaction.TransactionGuid);
transaction.TransactionName = "In-Game Purchase";
transaction.TransactionNameSize = strlen(transaction.TransactionName);
transaction.TransactionDateEpoch = getCurrentEpochTime();
transaction.TransactionValueUSD = 9.99f;

playerInfo.transactions = &transaction;
playerInfo.transactionsSize = 1;

int result = UpdatePlayer(28, "your-private-key", strlen("your-private-key"), playerInfo);
```
* `playerGuid` - Your player Id. Max 36 chars. **(Mandatory field)**
* `playerNickname` - Nickname of the player. Max 36 chars. **(optional field)**
* `playerEmail` - Email of the player. Max 36 chars. **(optional field)**
* `playerRank` - Integer rank of the player **(optional field)**
* `playerJoinDateEpoch` - Date when the player joined **(optional field)**
* `playerSuspectScore` - A manually assigned number, between 0-100, that can be updated to trigger rules you define in Getgud. **(optional field)**
* `playerReputation` - A field for you to use according to Rules you define, allowing you to set and continuously adjust Player Reputation to all your players - automatically. Max 36 chars. **(optional field)**
* `playerStatus` - A field for you to use according to Rules you define, allowing you to set and continuously adjust Player Status to all your players - automatically. Max 36 chars. **(optional field)**
* `PlayerCampaign` - The campaign to attribute this player to. Max 128 chars. **(optional field)**
* `playerDevice` - The device of the player. Max 8 chars. **(optional field)**
* `playerOS` - The OS of the player. Max 8 chars. **(optional field)**
* `playerAge` - The age of the player 0-128 **(optional field)**
* `playerGender` - The gender of the player. Max 8 chars. **(optional field)**
* `playerLocation` - The location of the player. Max 36 chars. **(optional field)**
* `playerNotes` - Any player notes you wish to save. Max 128 chars. **(optional field)**
* `transactions` - Transactions of the player **(optional field)**

Each item of `PlayerTransactions` has the following format:

```c
struct PlayerTransactions {
    const char* TransactionGuid; // Mandatory: Unique identifier for the transaction. Max 36 chars.
    int TransactionGuidSize;
    const char* TransactionName; // Mandatory: Name or description of the transaction. Max 36 chars.
    int TransactionNameSize;
    long long TransactionDateEpoch; // Optional: Date of the transaction in epoch time
    float TransactionValueUSD; // Optional: Value of the transaction in USD
};
```

## Configuration and Logging

The default Config file is loaded during `init()` and is located in the same directory as the SDK files. You can specify a custom configuration file path or pass the configuration content directly using the other `init` methods.

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
  "flushTimeoutMilliseconds": 10000,
  "logLevel": "FULL"
}
```

### Logging

The SDK logs its operations to a file named `logs.txt`. This file is created in the same directory as the SDK executable by default. You can control logging behavior through the following configuration parameters:

- `logToFile`: Set to `true` to enable logging to file, `false` to disable.
- `logFileSizeInBytes`: Maximum size of the log file before it's rotated (default: 10 MB).
- `circularLogFile`: If `true`, the log file will be overwritten from the beginning when it reaches the maximum size. If `false`, a new log file will be created.
- `logLevel`: Controls the verbosity of logging. Options are "FULL", "DEBUG", "INFO", "WARNING", "ERROR", and "FATAL".

To change the location of the log file, you can set the `GETGUD_LOG_FILE_PATH` environment variable to the desired file path.

## Examples

An example of how to use the GetGud C SDK can be found in the [examples](https://github.com/getgud-io/cpp-getgud-sdk-dev/tree/main/examples) directory.
