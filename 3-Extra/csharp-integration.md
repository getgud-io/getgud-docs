# Getgud.io C# SDK

## What Can You Do With Getgud's SDK:

Getgud's C# SDK allows you to integrate your game with the Getgud platform. Once integrated, you will be able to:

- Stream live game data to Getgud's cloud, including in-match actions, in-match reports, and in-match chat messages.
- Send Reports about historical matches to Getgud.
- Send (and update) player information to Getgud.

## Getting Started

Set the environment variables `GETGUD_TITLE_ID` and `GETGUD_PRIVATE_KEY` with the Title Id and Private Key you received from Getgud.io, or use them as arguments for StartGame. 

Include the following namespace:

```csharp
using GetgudSDK;
```

Initialize the SDK:

```csharp
int result = GetgudSDK.Methods.Init();
```

Start a Game:

```csharp
string gameGuid;
GetgudSDK.StartGameInfo gameInfo = new GetgudSDK.StartGameInfo
{
    ServerGuid = "cs2-server-1", // max 36 chars
    GameMode = "deathmatch", // max 36 chars
    ServerLocation = "203.0.113.42" // max 36 chars
};

int result = GetgudSDK.Methods.StartGame(gameInfo, out gameGuid);
```

Once the Game starts, you'll receive the Game's guid, now you can start a Match:

```csharp
string matchGuid;
GetgudSDK.StartMatchInfo matchInfo = new GetgudSDK.StartMatchInfo
{
    gameGuid = gameGuid,
    matchMode = "Knives-only", // max 36 chars
    mapName = "de-dust" // max 36 chars
};

int result = GetgudSDK.Methods.StartMatch(matchInfo, out matchGuid);
```

Once you create a match, you can send Actions to it.
Let's create a spawn action and send it to the match:

```csharp
GetgudSDK.SendSpawnActionInfo spawnInfo = new GetgudSDK.SendSpawnActionInfo
{
    baseData = new GetgudSDK.BaseActionData
    {
        actionTimeEpoch = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),
        matchGuid = matchGuid,
        playerGuid = "player-10"
    },
    characterGuid = "character-1",
    teamGuid = "team-1",
    initialHealth = 100f,
    position = new GetgudSDK.PositionF { X = 1f, Y = 2f, Z = 3f },
    rotation = new GetgudSDK.RotationF { Yaw = 10f, Pitch = 20f, Roll = 0f }
};

int result = GetgudSDK.Methods.SendSpawnAction(spawnInfo);
```

End a game (All Game's Matches will close as well):

```csharp
int result = GetgudSDK.Methods.MarkEndGame(gameGuid);
```

Flush and wait for all queued actions to be sent (optional, before shutdown):

```csharp
int flushResult = GetgudSDK.Methods.Flush();
```

Close and Dispose of the SDK:

```csharp
GetgudSDK.Methods.dispose();
```

## SDK Methods

### Init()

Initializes the SDK. You can call this method in three ways:

```csharp
static public int Init();
static public int InitConfPath(string configFilePath);
static public int InitConf(string configFile, int passAsContent);
```

- The first version uses default configuration.
- The second version allows you to specify a path to a configuration file.
- The third version allows you to pass the configuration content directly (if `passAsContent` is true) or as a file path (if `passAsContent` is false).

### StartGame(StartGameInfo info, out string gameGuidOut)

To start a new game, call `StartGame()`, this will use the environment variables `GETGUD_TITLE_ID` and `GETGUD_PRIVATE_KEY`.
`StartGame` returns the size of the game guid and outputs the `gameGuid` which you will use later to start new Matches inside the Game as well as to end the Game when it is over.

```csharp
static public int StartGame(StartGameInfo info, out string gameGuidOut);
```
* `info` - A structure containing game information:
  * `ServerGuid` - a unique name of your game server - String, Alphanumeric, max 36 chars.
  * `GameMode` - the mode of the game (each mode has a separate AI learning processes) - String, Alphanumeric, max 36 chars.
  * `ServerLocation` - a name/ip of your game server's location - String, Alphanumeric, max 36 chars.
* `gameGuidOut` - An output parameter to store the game guid.

### StartMatch(StartMatchInfo info, out string matchGuidOut)

Once you've started a live Game, you can now attach Matches to that Game.
When a live Match starts it returns a `matchGuid`, which is used to add Actions, Chat messages, and Reports to that specific match.

```csharp
static public int StartMatch(StartMatchInfo info, out string matchGuidOut);
```
* `info` - A structure containing match information:
  * `gameGuid` - The Game guid you received when starting a new Game
  * `matchMode` - the mode of the match (each mode has an additional AI learning processes) - String, Alphanumeric, max 36 chars.
  * `mapName` - the unique name of the map - String, Alphanumeric, max 36 chars.
* `matchGuidOut` - An output parameter to store the match guid.

### MarkEndGame(string gameGuid)

Ends a live Game and its associated matches.
When the live Game ends, you should mark it as finished in order to close it on Getgud's side.
Once the game is marked as ended, it will not accept new live data.

```csharp
static public int MarkEndGame(string gameGuid);

// Usage:
int result = GetgudSDK.MarkEndGame(gameGuid);
```
* `gameGuid` - The Game guid you received when starting a new Game

`MarkEndGame` returns an integer indicating success (1) or failure (0).

### Flush()

Waits until all queued actions are sent to the server before returning. Use this when you need to ensure all data has been transmitted before shutting down.

```csharp
static public int Flush();

// Usage:
int result = GetgudSDK.Flush();
```

`Flush` returns `1` if all queued actions were successfully sent, or `0` if the timeout was reached. The timeout is configured via `flushTimeoutMilliseconds` in config (default: 10 seconds).

## Sending Actions 

When a live Match starts, you can add Actions, Chat Data, and Reports to it. 
There are 7 Action types you can add to the Match, all derived from base action which holds the actions base properties.

### Base Action

`BaseActionData` is the base structure that all actions use and holds common information for all actions.

```csharp
public struct BaseActionData
{
    public long actionTimeEpoch;
    public string matchGuid;
    public string playerGuid;
};
```
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `matchGuid` - guid of the live Match where the action happened, is given to you when `StartMatch` is called. Max 36 chars.
* `playerGuid` - your player Id (which will be called playerGuid on Getgud's side). Max 36 chars.

### Spawn Action

A Spawn action should be sent whenever a player appears or reappears in-match, on the map.
To create a Spawn Action, use the `SendSpawnAction` method.

```csharp
static public int SendSpawnAction(SendSpawnActionInfo info);
```
* `info` - A structure containing spawn action information:
  * `baseData` - See BaseActionData
  * `characterGuid` - Guid of the spawned character from your game. Max 36 chars.
  * `teamGuid` - The identifier of the player's team. Max 36 chars.
  * `initialHealth` - The initial health of the player
  * `position` - X,Y,Z coordinates of the player at the moment of action.
  * `rotation` - YAW, PITCH, ROLL rotation of player view at the moment of action.

### Position Action

A Position action should be sent on player position change (including looking direction).
To create a Position Action, use the `SendPositionAction` method.

```csharp
static public int SendPositionAction(SendPositionActionInfo info);
```
* `info` - A structure containing position action information:
  * `baseData` - See BaseActionData
  * `position` - X,Y,Z coordinates of the player at the moment of action.
  * `rotation` - YAW, PITCH, ROLL rotation of player view at the moment of action.

### Attack Action

An Attack action is any attempt to create damage, now or in the future, for example, firing a shot, swinging a sword, placing a bomb, throwing a grenade, or any other action that may result in damage.
To create an Attack Action, use the `SendAttackAction` method.

```csharp
static public int SendAttackAction(SendAttackActionInfo info);
```
* `info` - A structure containing attack action information:
  * `baseData` - See BaseActionData
  * `weaponGuid` - A unique name of the weapon that the attack was performed with. Max 36 chars.

### Affect Action

An Affect action should be sent whenever an in-match affect of any kind is applied to the player. Examples: crouch, prone, jump, fly, use special ability, boost speed/ammo/shield/health, etc. 
To create an Affect Action, use the `SendAffectkAction` method.

```csharp
static public int SendAffectkAction(SendAffectActionInfo info);
```
* `info` - A structure containing affect action information:
  * `baseData` - See BaseActionData
  * `affectGuid` - A unique name of the affect. Max 36 chars.
  * `affectState` - An enumeration unit that allows you to flag the state of the Affect:
    * Attach - indicate affect is attached to a player (not mandatory).
    * Detach - indicate affect is detached from a player (not mandatory).
    * Activate - indicate affect is affecting the player.
    * Deactivate - indicate affect stopped affecting the player.

### Damage Action

A Damage action should be raised when a player loses health, both PVP and PVE.
To create a Damage Action, use the `SendDamageAction` method.

```csharp
static public int SendDamageAction(SendDamageActionInfo info);
```
* `info` - A structure containing damage action information:
  * `baseData` - See BaseActionData
  * `victimPlayerGuid` - guid of the player who is the victim of the Damage action. Max 36 chars.
  * `damageDone` - How much damage was given
  * `weaponGuid` - A unique name of the weapon that the attack was performed with. Max 36 chars.

### Heal Action

A Heal action should be triggered whenever a player is healed, doesn't matter by whom or how.
To create a heal action, use the `SendHealAction` method.

```csharp
static public int SendHealAction(SendHealActionInfo info);
```
* `info` - A structure containing heal action information:
  * `baseData` - See BaseActionData
  * `healthGained` - How much health the player gained.

### Death Action

A Death action should be triggered on a death of a player, either by another player, the environment or the player itself.
To create a death action to a match, use the `SendDeathAction` method.

```csharp
static public int SendDeathAction(SendDeathActionInfo info);
```
* `info` - A structure containing death action information:
  * `baseData` - See BaseActionData
  * `attackerGuid` - guid of the player who killed the victim of the Damage action. Max 36 chars.

## Adding Chat Messages
To add a chat message to a live Match:

```csharp
ChatMessageInfo messageInfo = new ChatMessageInfo
{
    messageTimeEpoch = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),
    matchGuid = matchGuid,
    playerGuid = "player1",
    message = "Hi from Getgud!"
};

int result = GetgudSDK.Methods.SendChatMessage(messageInfo);
```
* `messageTimeEpoch` - epoch time in milliseconds when the message was sent
* `matchGuid` - The guid of the match where the message was sent. Max 36 chars.
* `playerGuid` - The guid of the player who sent the message. Max 36 chars.
* `message` - The content of the chat message. Max 256 chars.

## Adding Reports

To add a report to a live Match:
Note that all of the fields are optional except `matchGuid`, `reportedTimeEpoch`, and `suspectedPlayerGuid` (a report must have a valid match, time, and player).
This is an async method which will not block the calling thread.

```csharp
ReportInfo reportInfo = new ReportInfo(
    matchGuid: "549cf69d-0d55-4849-b2d1-a49a4f0a0b1e",
    reportedTimeEpoch: DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),
    suspectedPlayerGuid: "player1",
    reporterName: "player2",
    reporterType: 1, // Moderator
    reporterSubType: 1, // QA
    tbType: 1, // Wallhack
    tbTimeEpoch: DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),
    suggestedToxicityScore: 100
);

int result = GetgudSDK.Methods.SendInMatchReport(reportInfo);
```
* `matchGuid` - guid of the live Match you are sending a report to. Max 36 chars. **(Mandatory field)**
* `reportedTimeEpoch` - epoch time in milliseconds of when the report was created **(Mandatory field)**
* `suspectedPlayerGuid` - the player Id of the suspected player as appears in your system. Max 36 chars. **(Mandatory field)**
* `reporterName` - the name of the entity that created the report. Max 36 chars. **(optional field)**
* `reporterType` - the type of the entity that created the report **(optional field)**
* `reporterSubType` - the subtype of the entity that created the report **(optional field)**
* `tbType` - id of the toxic behavior type, for example, Aimbot **(optional field)**
* `tbTimeEpoch` - epoch time of when the toxic behavior event occurred **(optional field)**
* `suggestedToxicityScore` - 0-100 toxicity score, ie: how much do you suspect the player **(optional field)**

### Sending Reports On Historical Matches

To add reports to a historical Match (a match which is not live and has ended):

```csharp
static public int SendReport(int titleId, string privateKey, ReportInfo report);
```

## Sending Player Updates

To update player information, you can call `UpdatePlayer`:
All fields are optional except player Id.
This is an async method which will not block the calling thread.

```csharp
PlayerInfo playerInfo = new PlayerInfo(
    playerGuid: "549cf69d-0d55-4849-b2d1-a49a4f0a0b1e",
    playerNickname: "test",
    playerEmail: "test@test.com",
    playerRank: 10,
    playerJoinDateEpoch: DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),
    playerSuspectScore: "12",
    playerReputation: "Toxic",
    playerStatus: "Banned",
    playerCompaign: "Tiktok-2023-Feb-23-Id1332",
    playerDevice: "PC",
    playerOS: "Win11",
    playerAge: 27,
    playerGender: "Male",
    playerLocation: "203.0.113.42",
    playerNotes: "I just banned this player from my system because of XYZ"
);

// Add player transaction
playerInfo.transactions.Add(new PlayerTransactionsData
{
    TransactionGuid = "trans-001",
    TransactionName = "In-Game Purchase",
    TransactionDateEpoch = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),
    TransactionValueUSD = 9.99f
});

int result = GetgudSDK.Methods.UpdatePlayer(28, "your-private-key", playerInfo);
```
* `PlayerGuid` - Your player Id. Max 36 chars. **(Mandatory field)**
* `PlayerNickname` - Nickname of the player. Max 36 chars. **(optional field)**
* `PlayerEmail` - Email of the player. Max 36 chars. **(optional field)**
* `PlayerRank` - Integer rank of the player **(optional field)**
* `PlayerJoinDateEpoch` - Date when the player joined **(optional field)**
* `PlayerSuspectScore` - A manually assigned number, between 0-100, that can be updated to trigger rules you define in Getgud. **(optional field)**
* `PlayerReputation` - A field for you to use according to Rules you define, allowing you to set and continuously adjust Player Reputation to all your players - automatically. Max 36 chars. **(optional field)**
* `PlayerStatus` - A field for you to use according to Rules you define, allowing you to set and continuously adjust Player Status to all your players - automatically. Max 36 chars. **(optional field)**
* `PlayerCampaign` - The campaign to attribute this player to. Max 128 chars. **(optional field)**
* `PlayerDevice` - The device of the player. Max 8 chars. **(optional field)**
* `PlayerOS` - The OS of the player. Max 8 chars. **(optional field)**
* `PlayerAge` - The age of the player 0-128 **(optional field)**
* `PlayerGender` - The gender of the player. Max 8 chars. **(optional field)**
* `PlayerLocation` - The location of the player. Max 36 chars. **(optional field)**
* `PlayerNotes` - Any player notes you wish to save. Max 128 chars. **(optional field)**
* `Transactions` - Transactions of the player **(optional field)**

Each item of `PlayerTransactions` has the following format:

```csharp
public struct PlayerTransactionsData
{
    public string TransactionGuid; // Mandatory: Unique identifier for the transaction. Max 36 chars.
    public string TransactionName; // Mandatory: Name or description of the transaction. Max 36 chars.
    public long TransactionDateEpoch; // Optional: Date of the transaction in epoch time
    public float TransactionValueUSD; // Optional: Value of the transaction in USD
};
```

## Configuration and Logging

The default Config file is loaded during `Init()` and is located in the same directory as the SDK files. You can specify a custom configuration file path or pass the configuration content directly using the other `Init` methods.

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

## Dispose

When you're done using the SDK, make sure to dispose of it:

```csharp
GetgudSDK.Methods.dispose();
```

## Examples

An example of how to use the Getgud C# SDK can be found in the [examples](https://github.com/getgud-io/cpp-getgud-sdk-dev/tree/main/examples) directory.
