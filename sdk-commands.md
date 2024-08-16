# Getgud.io C++ SDK

Same commands are available for C++, C#, C and Python languages.

* [C++ header file](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/include/GetgudSDK.h)
* [C header file](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/include/GetgudSDK_C.h)
* [Python wrapper around C header](https://github.com/getgud-io/cpp-getgud-sdk-dev/blob/main/examples/python/getgudsdk_wrapper.py)

## What Can You Do With Getgud's SDK:

Getgud's C++ SDK allows you to integrate your game with the GetGud platform. Once integrated, you will be able to:

- Stream live game data to GetGud's cloud, including in-match actions, in-match reports, and in-match chat messages.
- Send Reports about historical matches to Getgud.
- Send (and update) player information to Getgud.

## Getting Started

Set the environment variables `GETGUD_TITLE_ID` and `GETGUD_PRIVATE_KEY` with the Title Id and Private Key you received from Getgud.io, or use them as arguments for StartGame. (multiple title support available below)

Include the following header file:

```cpp
#include <GetgudSDK.h>
```

Initialize the SDK:

```cpp
GetgudSDK::Init();
```

Start a Game:

```cpp
std::string gameGuid = GetgudSDK::StartGame(
  "cs2-server-1", // serverGuid
  "deathmatch", // gameMode
  "203.0.113.42" // serverLocation
);
```

Once the Game starts, you'll receive the Game's guid, now you can start a Match:

```cpp
std::string matchGuid = GetgudSDK::StartMatch(
  gameGuid, 
  "Knives-only", // matchMode
  "de-dust" // mapName
);
```

Once you create a match, you can send Actions to it.
Let's create a deque of Actions and send it to the match:

```cpp
//Create a deque of Actions
std::deque<GetgudSDK::BaseActionData*> actionsToSend {
  new GetgudSDK::SpawnActionData(
    matchGuid, curTimeEpoch, "player-10", "character-1", "team-1", 100.f,
    GetgudSDK::PositionF{1, 2, 3}, GetgudSDK::RotationF{10, 20, 0}),
			
  new GetgudSDK::PositionActionData(
    matchGuid, curTimeEpoch, "player-5",
    GetgudSDK::PositionF{20.32000f, 50.001421f, 0.30021f},
    GetgudSDK::RotationF{10, 20, 0})
};

//Send it to the SDK
GetgudSDK::SendActions(actionsToSend);

//Clean up the actions
for (auto* sentAction : actionsToSend)
    delete sentAction;
actionsToSend.clear();
```

End a game (All Game's Matches will close as well):

```cpp
bool gameEnded = GetgudSDK::MarkEndGame(gameGuid);
```

Close and Dispose of the SDK:

```cpp
GetgudSDK::Dispose();
```

## SDK Methods

### Init()

Initializes the SDK. You can call this method in three ways:

```cpp
bool Init();
bool Init(std::string configFileFullPath);
bool Init(std::string configFile, bool passAsContent);
```

- The first version uses default configuration.
- The second version allows you to specify a path to a configuration file.
- The third version allows you to pass the configuration content directly (if `passAsContent` is true) or as a file path (if `passAsContent` is false).

### StartGame(serverGuid, gameMode, serverLocation)

To start a new game, call `StartGame()`, this will use the environment variables `GETGUD_TITLE_ID` and `GETGUD_PRIVATE_KEY`.
`StartGame` returns `gameGuid` - a unique identifier of the game which you will use later to start new Matches inside the Game as well as to end the Game when it is over.

```cpp
std::string gameGuid = GetgudSDK::StartGame(serverGuid, gameMode, serverLocation);
```
* `serverGuid` - a unique name of your game server - String, Alphanumeric, 36 chars max.
* `gameMode` - the mode of the game (each mode has a separate AI learning processes) - String, Alphanumeric, 36 chars max.
* `serverLocation` - a name/ip of your game server's location - String, Alphanumeric, 36 chars max.

#### StartGame(titleId, privateKey, serverGuid, gameMode, serverLocation)

You can also call the `StartGame()` method with titleId and privateKey that you pass (supporting multiple titles on the same machine)

```cpp
std::string gameGuid = GetgudSDK::StartGame(titleId, privateKey, serverGuid, gameMode, serverLocation);
```
* `titleId` - The title Id you received from Getgud.io
* `privateKey` - The Private Key for the above title you received together with its Title Id
* `serverGuid` - a unique name of your game server - String, Alphanumeric, 36 chars max.
* `gameMode` - the mode of the game (each mode has an additional AI learning processes) - String, Alphanumeric, 36 chars max.
* `serverLocation` - a name/ip of your game server's location - String, Alphanumeric, 36 chars max.

### StartMatch(gameGuid, matchMode, mapName)

Once you've started a live Game, you can now attach Matches to that Game.
When a live Match starts it returns a `matchGuid`, which is used to add Actions, Chat messages, and Reports to that specific match.
To start a new match, call `StartMatch()`:

```cpp
std::string matchGuid = GetgudSDK::StartMatch(gameGuid, matchMode, mapName);
```
* `gameGuid` - The Game guid you received when starting a new Game
* `matchMode` - the mode of the match (each mode has an additional AI learning processes) - String, Alphanumeric, 36 chars max.
* `mapName` - the unique name of the map - String, Alphanumeric, 36 chars max.

### SendActions(std::deque<BaseActionData*> actions)

Once you've started a Match, you can now send actions to it.
To add a batch of actions to a match, use the `SendActions` function.
This is an async method which will not block the calling thread.

```cpp
bool SendActions(std::deque<BaseActionData*> actions);
```
* `actions` - deque of `BaseActionData` objects, where `BaseActionData` is the base class of all the primal 7 actions (Spawn, Position, Attack, Damage, Heal, Affect and Death).

#### SendAction(BaseActionData* action)

You can also send single actions when you need to, using the `SendAction()` method.
This is an async method which will not block the calling thread.

```cpp
bool SendAction(BaseActionData* action);
```
* `action` - a `BaseActionData` object that is the base class of all the primal 7 actions (Spawn, Position, Attack, Damage, Heal, Affect and Death).

### MarkEndGame(gameGuid)

Ends a live Game and its associated matches.
When the live Game ends, you should mark it as finished in order to close it on Getgud's side.
Once the game is marked as ended, it will not accept new live data.

```cpp
bool gameEnded = GetgudSDK::MarkEndGame(gameGuid);
```
* `gameGuid` - The Game guid you received when starting a new Game

`MarkEndGame` returns true/false depending if the Game was successfully closed or not.

## Sending Actions 

When a live Match starts, you can add Actions, Chat Data, and Reports to it. 
There are 7 Action types you can add to the Match, all derived from base action which holds the actions base properties.

### Base Action

`BaseAction` is the base class that all actions derive from and holds common information to all actions.

```cpp
BaseAction(std::string matchGuid,
        long long actionTimeEpoch,
        std::string playerGuid);
```
* `matchGuid` - guid of the live Match where the action happened, is given to you when `StartMatch` is called.
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `playerGuid` - your player Id (which will be called playerGuid on Getgud's side) 

### Spawn Action

A Spawn action should be sent whenever a player appears or reappears in-match, on the map.
To create a Spawn Action, use the `SpawnActionData` class.
This action marks the appearance or reappearance of a `Player` inside the Match.

```cpp
GetgudSDK::BaseActionData* action = new SpawnActionData(
        std::string matchGuid,
        long long actionTimeEpoch,
        std::string playerGuid,
        std::string characterGuid,
        std::string teamGuid,
        float initialHealth,
        PositionF position,
        RotationF rotation);
```
* `matchGuid` - guid of the live Match where the action happened
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `playerGuid` - your player Id
* `characterGuid` - Guid of the spawned character from your game, 36 max length.
* `teamGuid` - The identifier of the player's team
* `initialHealth` - The initial health of the player
* `position` - X,Y,Z coordinates of the player at the moment of action.
* `rotation` - YAW, PITCH, ROLL rotation of player view at the moment of action.

### Position Action

A Position action should be sent on player position change (including looking direction).
To create a Position Action, use the `PositionActionData` class.
This action marks the change of `Player` position and view site.

```cpp
GetgudSDK::BaseActionData* action = new GetgudSDK::PositionActionData(
                     std::string matchGuid,
                     long long actionTimeEpoch,
                     std::string playerGuid,
                     PositionF position,
                     RotationF rotation
);
```
* `matchGuid` - guid of the live Match where the action happened
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `playerGuid` - your player Id
* `Position` - X,Y,Z coordinates of the player at the moment of action.
* `Rotation` - YAW, PITCH, ROLL rotation of player view at the moment of action.

### Attack Action

An Attack action is any attempt to create damage, now or in the future, for example, firing a shot, swinging a sword, placing a bomb, throwing a grenade, or any other action that may result in damage.
To create an Attack Action, use the `AttackActionData` Class.
Note that the Attack action is not bound to the `Damage` action, it is an attempt to cause Damage, not the Damage event itself.

```cpp
GetgudSDK::BaseActionData* action = new GetgudSDK::AttackActionData(
                     std::string matchGuid,
                     long long actionTimeEpoch,
                     std::string playerGuid,
                     std::string weaponGuid
);
```
* `matchGuid` - guid of the live Match where the action happened
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `playerGuid` - your player Id
* `weaponGuid` - A unique name of the weapon that the attack was performed with, max length is 36 chars.

### Affect Action

An Affect action should be sent whenever an in-match affect of any kind is applied to the player. Examples: crouch, prone, jump, fly, use special ability, boost speed/ammo/shield/health, etc. 
To create an Affect Action, use the `AffectActionData` Class.

```cpp
GetgudSDK::BaseActionData* action = new GetgudSDK::AffectActionData(
                     std::string matchGuid,
                     long long actionTimeEpoch,
                     std::string playerGuid,
                     std::string affectGuid,
                     AffectState affectState
);
```
* `matchGuid` - guid of the live Match where the action happened
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `playerGuid` - your player Id
* `affectGuid` - A unique name of the affect, max length is 36 chars.
* `affectState` - An enumeration unit that allows you to flag the state of the Affect:
  	Attach - indicate affect is attached to a player (not mandatory).
  	Detach - indicate affect is detached from a player (not mandatory).
  	Activate - indicate affect is affecting the player.
  	Deactivate - indicate affect stopped affecting the player.  

### Damage Action

A Damage action should be raised when a player loses health, both PVP and PVE. If the Damage is caused by the environment you can specify this in a playerGuid using a predefined variable `GetgudSDK::Values::Environment`
To create a Damage Action, use the `DamageActionData` class. 

```cpp
GetgudSDK::BaseActionData* action = new GetgudSDK::DamageActionData(
                     std::string matchGuid,
                     long long actionTimeEpoch,
                     std::string playerGuid,
                     std::string victimPlayerGuid,
                     float damageDone,
                     std::string weaponGuid
);
```
* `matchGuid` - guid of the live Match where the action happened
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `playerGuid` - your player Id
* `victimPlayerGuid` - guid of the player who is the victim of the Damage action, max length is 36 chars.
* `damageDone` - How much damage was given
* `weaponGuid` - A unique name of the weapon that the attack was performed with, max length is 36 chars.

### Heal Action

A Heal action should be triggered whenever a player is healed, doesn't matter by whom or how.
To create a heal action, use the `HealActionData` class.
```cpp
GetgudSDK::BaseActionData* action = new GetgudSDK::HealActionData(
                     std::string matchGuid,
                     long long actionTimeEpoch,
                     std::string playerGuid,
                     float healthGained
);
```
* `matchGuid` - guid of the live Match where the action happened
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `playerGuid` - your player Id
* `healthGained` - How much health the player gained.

### Death Action

A Death action should be triggered on a death of a player, either by another player, the environment or the player itself.
To create a death action to a match, use the `DeathActionData` class. 

```cpp
GetgudSDK::BaseActionData* action = new GetgudSDK::DeathActionData(
                     std::string matchGuid,
                     long long actionTimeEpoch,
                     std::string playerGuid,
                     std::string attackerGuid
);
```
* `matchGuid` - guid of the live Match where the action happened
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `playerGuid` - your player Id
* `attackerGuid` - guid of the player who killed the victim of the Damage action, max length is 36 chars. Use `GetgudSDK::Values::Environment` to indicate that PvE killed the player

## Adding Chat Messages
To add a chat message to a live Match:

```cpp
GetgudSDK::ChatMessageInfo messageData;
messageData.message = "Hi from Getgud!";
messageData.messageTimeEpoch = 1684059337532;
messageData.playerGuid = "player1";
GetgudSDK::SendChatMessage(
  "6a3d1732-8f72-12eb-bdef-56d89392f384", // matchGuid 
  messageData
);
```

## Adding Reports

To add a report to a live Match:
Note that all of the fields are optional except `MatchGuid`, `ReportedTimeEpoch`, and `SuspectedPlayerGuid` (a report must have a valid match, time, and player).<br> 
To fill `ReporterType`, `ReporterSubType` and `TbType` fields you can use enums exposed to you by our SDK.
This is an async method which will not block the calling thread.

```cpp
GetgudSDK::ReportInfo reportInfo;
reportInfo.MatchGuid = "549cf69d-0d55-4849-b2d1-a49a4f0a0b1e";
reportInfo.ReportedTimeEpoch = 1684059337532;
reportInfo.ReporterName = "player1";
reportInfo.ReporterType = GetgudSDK::ReporterType::Moderator;
reportInfo.ReporterSubType = GetgudSDK::ReporterSubtype::QA;
reportInfo.SuggestedToxicityScore = 100;
reportInfo.SuspectedPlayerGuid = "player1";
reportInfo.TbTimeEpoch = 1684059337532;
reportInfo.TbType = GetgudSDK::TbType::Wallhack;
GetgudSDK::SendInMatchReport(reportInfo);
```
* `MatchGuid`- guid of the live Match you are sending a report to **(Mandatory field)**
* `ReportedTimeEpoch`- epoch time of when the report was created **(Mandatory field)**
* `ReporterName`- the name of the entity that created the report **(optional field)**
* `ReporterType`- the type of the entity that created the report **(optional field)**
* `ReporterSubType`-   the subtype of the entity that created the report **(optional field)**
* `SuggestedToxicityScore`- 0-100 toxicity score, ie: how much do you suspect the player **(optional field)**
* `SuspectedPlayerGuid`- the player Id of the suspected player as appears in your system **(Mandatory field)**
* `TbType` - id of the toxic behavior type, for example, Aimbot **(optional field)**
* `TbTimeEpoch` - epoch time of when the toxic behavior event occurred **(optional field)**

### Sending Reports On Historical Matches

To add reports to a historical Match (a match which is not live and has ended):

```cpp
std::deque<GetgudSDK::ReportInfo> reports;
GetgudSDK::ReportInfo reportInfo;
reportInfo.MatchGuid = "549cf69d-0d55-4849-b2d1-a49a4f0a0b1e";
reportInfo.ReportedTimeEpoch = 1684059337532;
reportInfo.ReporterName = "player1";
reportInfo.ReporterSubType = GetgudSDK::ReporterSubtype::QA;
reportInfo.ReporterType = GetgudSDK::ReporterType::Moderator;
reportInfo.SuggestedToxicityScore = 100;
reportInfo.SuspectedPlayerGuid = "player1";
reportInfo.TbTimeEpoch = 1684059337532;
reportInfo.TbType = GetgudSDK::TbType::Wallhack;
reports.push_back(reportInfo);

GetgudSDK::SendReports(
  28, // titleId
  "6a3d1732-8f72-12eb-bdef-56d89392f384",  // privateKey
  reports
);
```
You can also use SendReports function without `titleId` and `privateKey` arguments, in case you have `GETGUD_TITLE_ID` and `GETGUD_PRIVATE_KEY` env variables defined.

Deque of reports:

```cpp
GetgudSDK::SendReports(reports);
```

## Sending Player Updates

To update player information, you can call `UpdatePlayers`:
All fields are optional except player Id.
Use a deque of PlayerInfo objects to send to Getgud SDK.
This is an async method which will not block the calling thread.

```cpp
std::deque<GetgudSDK::PlayerInfo> playerInfos;
GetgudSDK::PlayerInfo playerInfo;
playerInfo.PlayerGuid = "549cf69d-0d55-4849-b2d1-a49a4f0a0b1e";
playerInfo.PlayerNickname = "test";
playerInfo.PlayerEmail = "test@test.com";
playerInfo.PlayerRank = 10;
playerInfo.PlayerSuspectScore = "12";
playerInfo.PlayerReputation = "Toxic";
playerInfo.PlayerStatus = "Banned";
playerInfo.PlayerCampaign = "Tiktok-2023-Feb-23-Id1332";
playerInfo.PlayerDevice = "PC";
playerInfo.PlayerOS = "Win11";
playerInfo.PlayerAge = 27;
playerInfo.PlayerGender = "Male";
playerInfo.PlayerLocation = "203.0.113.42";
playerInfo.PlayerNotes = "I just banned this player from my system because of XYZ";
playerInfo.PlayerJoinDateEpoch = 1684059337532;

// add player transaction
GetgudSDK::PlayerTransactions transaction;
transaction.TransactionGuid = "trans-001";
transaction.TransactionName = "In-Game Purchase";
transaction.TransactionDateEpoch = 1684059337532;
transaction.TransactionValueUSD = 9.99f;
playerInfo.Transactions.push_back(transaction);

playerInfos.push_back(playerInfo);
bool playersUpdated = GetgudSDK::UpdatePlayers(
  28, //titleId
  "6a3d1732-8f72-12eb-bdef-56d89392f384", // privateKey
  playerInfos
);
```
* `PlayerGuid`- Your player Id - String, 36 chars max. **(Mandatory field)**
* `PlayerNickname`- Nickname of the player - String, 36 chars max. **(optional field)**
* `PlayerEmail`- Email of the player **(optional field)** - String, 36 chars max.
* `PlayerRank`- Integer rank of the player **(optional field)**
* `PlayerSuspectScore`- A manually assigned number, between 0-100, that can be updated to trigger rules you define in Getgud. **(optional field)**
* `PlayerReputation`- A field for you to use according to Rules you define, allowing you to set and continuously adjust Player Reputation to all your players - automatically. - String, 36 chars max.**(optional field)**
* `PlayerStatus`- A field for you to use according to Rules you define, allowing you to set and continuously adjust Player Status to all your players - automatically. - String, 36 chars max.**(optional field)**
* `PlayerCampaign`- The campaign to attribute this player to - String, 128 chars max.**(optional field)**
* `PlayerDevice`-The device of the player - String, 8 chars max.**(optional field)**
* `PlayerOS`- The OS of the player - String, 8 chars max. **(optional field)**
* `PlayerAge`- The age of the player 0-128 **(optional field)**
* `PlayerGender`- The gender of the player - String, 8 chars max. **(optional field)**
* `PlayerLocation`- The location of the player - String, 36 chars max. **(optional field)**
* `PlayerNotes`- Any player notes you wish to save - String, 128 chars max. **(optional field)**
* `PlayerJoinDateEpoch`:  Date when the player joined **(optional field)**
* `Transactions`: Transactions of each player **(optional field)**


Each item of `PlayerTransactions` has the following format:

```cpp
struct PlayerTransactions {
    const char* TransactionGuid; // Mandatory: Unique identifier for the transaction
    const char* TransactionName; // Mandatory: Name or description of the transaction
    long long TransactionDateEpoch; // Optional: Date of the transaction in epoch time
    float TransactionValueUSD; // Optional: Value of the transaction in USD
};
```

You can use the `UpdatePlayers` function without `titleId` and `privateKey` arguments, in case you have `GETGUD_TITLE_ID` and `GETGUD_PRIVATE_KEY` env variables defined.

```cpp
bool playersUpdated = GetgudSDK::UpdatePlayers(playerInfos);
```

## Configuration

The default Config file is loaded during `GetgudSDK::Init();` and is located in the same dir as the SDK files.
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
  "logLevel": "FULL"
}
```

## Examples

An example of how to use the GetGud C++ SDK can be found in the [examples](https://github.com/getgud-io/cpp-getgud-sdk-dev/tree/main/examples) directory.
