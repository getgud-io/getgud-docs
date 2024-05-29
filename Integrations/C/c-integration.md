# GetGud C SDK

## What Can You Do With Getgud's SDK:

Getgud's C SDK allows you to integrate your game with the GetGud platform. Once integrated, you will be able to:

- Stream live game data to GetGud's cloud, including in-match actions, in-match reports, and in-match chat messages.
- Send Reports about historical matches to Getgud.
- Send (and update) player information to Getgud.

## Build requirements:
The shared and static libraries are different. The shared library can be linked directly to a project because it only requires the binaries and headers.
However, here is the way to link the static library correctly:
### Linux:
Dependencies:
- libcurl development kit.
- zlib development kit.
- openssl development kit.
- libssl development kit.
- libcrypto development kit.
- gcc packages.
- pthread library.

### Windows:
- libcurl library package.
- zlib library package.

A test project with completed setup is included in each release folder.

## Getting Started

Insert the Title Id and Private Key you recieved from Getgud.io to the `GETGUD_TITLE_ID ` and `GETGUD_PRIVATE_KEY` environment variables, or use them as arguments for StartGame. (multiple title support available below)

Include the follwing header file:

```c
#include <GetGudSdk.h>
```

Initialize the SDK:

```c
init();
```

Start a Game:

```c
char* gameGuid[32];
struct StartGameInfo gameInfo;
...
  fill the structure's fields.
...
int gameGuid_size = StartGame(
  gameInfo, // StartGameInfo
  gameGuid // output gameGuid
);
```

Once the Game starts, you'll recieve the Game's guid, now you can start a Match:

```c
struct StartMatchInfo matchInfo;
char* matchGuid[32];
...
  fill the structure's fields.
...
int matchGuidSize = GetGudSdk::StartMatch(
  matchInfo, //StartMatchInfo
  matchGuid // output matchGuid
);
```

Once you create a match, you can send Actions to it.
Let's create a deque of Actions and send it to the match:

```c
//Send an action
struct BaseActionData baseData;
...
  fill the structure's fields.
...
int isSent = SendAttackAction(baseData, // BaseActionData
                     "AVP", // weaponName
                     3); // weaponNameSize
```

End a game (All Game's Matches will close as well):

```c
int gameEnded = MarkEndGame(gameGuid, // started gameGuid
                         32); // gameGuidSize
```

Close and Dispose of the SDK:

```c
dispose();
```

## SDK Methods

### StartGame(struct StartMatchInfo, char gameGuid[32]) -> int gameGuidSize

To start a new game, call `StartGame()`, this will use the environment variables `TITLE_ID` and `PRIVATE_KEY`.
`StartGame` returns `gameGuid` - a unique identifier of the game which you will use later to start new Matches inside the Game as well as to end the Game when it is over.

```c
int gameGuid_size = StartGame(
  gameInfo, // StartGameInfo
  gameGuid // output gameGuid
);
```
* `gameInfo` - a qunique information of your game server and optional credentials.
* `gameGuid` - a 32-byte initialized pointer of chars that uses as an output parameter.

### StartMatch(struct StartMatchInfo, char matchGuid[32]) -> int gameGuidSize

Once you've started a live Game, you can now attach Matches to that Game.
When a live Match starts it returns a `matchGuid`, you'll need it to add Actions, Chat, and Reports to that match
To start a new match for a live game, call `StartMatch()`:

```c
int matchGuidSize = GetGudSdk::StartMatch(
  matchInfo, //StartMatchInfo
  matchGuid // output matchGuid
);
```
* `matchInfo` - A match information and the game guid you recieved when starting a new Game
* `matchGuid` - a 32-byte initialized pointer of chars that uses as an output parameter.

### MarkEndGame(gameGuid, gameGuidSize) -> int result

Ends a live Game and it's associated matches.
When the live Game ends, you should mark it as finished in order to close it on Getgud's side.
Once the game is marked as ended, it will not accept new live data.

```cpp
int gameEnded = MarkEndGame(gameGuid, 
  gameGuidSize);
```
* `gameGuid` - The Game guid you recieved when starting a new Game
* 'gameGuidSize' - The size of the received game guid. Should be equal 32.

`MarkEndGame` returns true/false depending if the Game was successfully closed or not.

## Creating Actions 

When a live Match starts, you can add Actions, Chat Data, and Reports to it. 
There are 6 Action types you can add to the Match, all derived from base action which holds the actions base properties.

### Base Action

`BaseAction` is the base class that all actions derive from and holds common information to all action.

```c
struct BaseActionData {
  long long actionTimeEpoch;
  char* matchGuid;
  int matchGuidSize;
  char* playerGuid;
  int playerGuidSize;
};
```
* `matchGuid` - guid of the live Match where the action happened, is given to you when `StartMatch` is called.
* `actionTimeEpoch` - epoch time in milliseconds when the action happened
* `playerId` - your player Id (which will called playerGuid on Getgud's side) 

### Spawn Action

To create a Spawn Action, use the `SpawnActionData` class. This action marks the appearance or reappearance of a `Player` inside the Match.

```c
int SendSpawnAction(struct BaseActionData baseData,
                    char* characterGuid,
                    int characterGuidSize,
                    int teamId,
                    float initialHealth,
                    struct PositionF position,
                    struct RotationF rotation);
```
* `BaseActionData` - See BaseActionData
* `characterGuid` - Guid of the spwaned character from your game, 36 max length.
* `teamId` - The identifier number of the player's team
* `initialHealth` - The initial health of the player
* `position` - X,Y,Z coordinates of the player at the moment of action.
* `rotation` - PITCH, ROLL rotation of player view at the moment of action.

### Position Action

To create a Position Action, use the `PositionActionData` class.
This action marks the change of `Player` position and view site.

```c
int SendPositionAction(struct BaseActionData baseData,
                       struct PositionF position,
                       struct RotationF rotation);
```
* `BaseAction` - See BaseAction
* `Position` - X,Y,Z coordinates of the player at the moment of action.
* `Rotation` - PITCH, ROLL rotation of player view at the moment of action.

### Attack Action

An Attack action is any attempt to create damage, now or in the future, for example, firing a shot, swinging a sword, placing a bomb, throwing a grenade, or any other action that may result in damage.
Note that the Attack action is not bound to the `Damage` action, it is an attempt to cause Damage, not the Damage event itself.

```c
int SendHealAction(struct BaseActionData baseData, float healthGained);
```
* `BaseAction` - See BaseAction
* `weaponGuid` - A unique name of the weapon that the attack was performed with, max length is 36 chars.

### Affect Action

An Affect action is any attempt to create an affect based on the states, now or in the future, for example, taking an item, obtaining a buff or joining into an aura.

```c
int SendPositionAction(struct BaseActionData baseData,
                      char* affectGuid,
                      int affectGuidSize,
                      AffectState affectState);
```
* `BaseAction` - See BaseAction
* `affectGuid` - A unique name of the affect, max length is 36 chars.
* `affectState` - An emumiration unit, Attach, Activate, Deactivate or Detach.

### Damage Action

To create a Damage Action, use the `DamageActionData` class. A Damage should be triggered when a player loses health, both PVP and PVE. If the Damage is caused by the environment you can specify this in a playerGuid using a predefined variable `GetGudSdk::Values::Environment`

```c
int SendDamageAction(struct BaseActionData baseData,
                     char* victimPlayerGuid,
                     int victimPlayerGuidSize,
                     float damageDone,
                     char* weaponGuid,
                     int weaponGuidSize);
```
* `BaseAction` - See BaseAction
* `victimPlayerGuid` - guid AKA nickname of the player who is the victim of the Damage action, max length is 36 chars.
* `damageDone` - How much damage was given
* `weaponGuid` - A unique name of the weapon that the attack was performed with, max length is 36 chars.

### Heal Action

To create a heal action, use the `HealActionData` class. A Heal action causes the player to increase his health while healing.

```c
int SendHealAction(struct BaseActionData baseData, float healthGained);
```
* `BaseAction` - See BaseAction
* `healthGained` - How much health the player gained.

### Death Action

To create a death action to a match, use the `DeathActionData` class. The Death player causes the player to Die, if the player died it should ALWAYS be marked. We do not detect this automatically.

```c
int SendDeathAction(struct BaseActionData baseData);
);
```
* `BaseAction` - See BaseAction

## Adding Chat Messages
To add a chat message to a live Match:

```c
struct ChatMessageInfo {
  long long messageTimeEpoch;
  char* matchGuid;
  int matchGuidSize;
  char* playerGuid;
  int playerGuidSize;
  char* message;
  int messageSize;
};

GetGudSdk::SendChatMessage(
  messageInfo // struct ChatMessageInfo 
);
```
* `messageTimeEpoch`- epoch time of when the message was sent **(Mandatory field)**
* `MatchGuid`- guid of the live Match you are sending a message to **(Mandatory field)**
* `playerGuid`- guid of the player that sent the message **(Mandatory field)**
* `message`- the message that was sent **(Mandatory field)**

## Adding Reports

To add a reports to a live Match:
Note that all of the fields are optional exept `MatchGuid` and `SuspectedPlayerId` (a report must have a valid match and player).<br> 
To fill `ReporterType`, `ReporterSubType` and `TbType` fields you can use enums exposed to you by our SDK.
This is an async method which will not block the calling thread.

```c
struct ReportInfo {
  char* matchGuid;
  int matchGuidSize;
  char* reporterName;
  int reporterNameSize;
  int reporterType;
  int reporterSubType;
  char* suspectedPlayerGuid;
  int suspectedPlayerGuidSize;
  int tbType;
  long long tbTimeEpoch;
  int suggestedToxicityScore;
  long long reportedTimeEpoch;
};

int SendInMatchReport(struct ReportInfo reportInfo);
```
* `MatchGuid`- guid of the live Match you are sending a report to **(Mandatory field)**
* `ReporterName`- the name of the entity that created the report **(optional field)**
* `ReporterType`- the type of the entity that created the report **(optional field)**
* `ReporterSubType`-   the subtype of the entity that created the report **(optional field)**
* `SuspectedPlayerId`- the player Id of the suspected player **(Mandatory field)**
* `TbType` - id of the toxic behavior type, for example, Aimbot **(optional field)**
* `TbTimeEpoch` - epoch time of when the toxic behavior event occured **(optional field)**
* `SuggestedToxicityScore`- 0-100 toxicity score, ie: how much do you suspect the player **(optional field)**
* `ReportedTimeEpoch`- epoch time of when the report was created **(Mandatory field)**

### Sending Reports On Historical Matches

To add a reports to a histotical Match (a match which is not live and has ended):

```c
int SendReport(int titleId,
  char* privateKey, int privateKeySize, struct ReportInfo report);
```

## Sending Player Updates

To update player information, you can call `UpdatePlayers`:
All fields are optional except player Id.
Use a deque of PlayerInfo objects to send to Getgud SDK.
This is an async method which will not block the calling thread.

```c
struct PlayerInfo {
  char* playerGuid;
  int playerGuidSize;
  char* playerNickname;
  int playerNicknameSize;
  char* playerEmail;
  int playerEmailSize;
  int playerRank;
  long long playerJoinDateEpoch;
};

int UpdatePlayer(int titleId,
  char* privateKey, int privateKeySize, struct PlayerInfo player);
```
* `playerGuid`- Your player Id - String, 36 chars max. **(Mandatory field)**
* `PlayerNickname`- Nickname of the player **(optional field)**
* `PlayerEmail`- Email of the player **(optional field)**
* `PlayerRank`- Integer rank of the player **(optional field)**
* `PlayerJoinDateEpoch`:  Date when the player joined **(optional field)**

## Configuration

The default Config file is loaded during `GetGudSdk::Init();` and is located in the same dir as the SDK files.
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

## Additional SDK Methods

The GetGud C SDK provides several other methods for sending chat messages, reports, updating player information, and more. Refer to the [SDK Events documentation](https://github.com/getgud-io/getgud-docs/blob/main/Integrations/C%2B%2B/cpp-integration.md#sdk-methods) and examples for detailed usage instructions of these methods.


## Examples

An example of how to use the GetGud C SDK can be found in the [examples](https://github.com/getgud-io/c-getgud-sdk/tree/main/examples) directory.
