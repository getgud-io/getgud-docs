# Getgud UE5 Blueprint Integration Tutorial

## Introduction

The Getgud Blueprint Plugin provides visual scripting nodes for integrating the GetgudSDK with Unreal Engine 5. This allows you to send player actions, game events, and reports to Getgud.io's cloud without writing C++ code.

For C++ integration, see the [C++ Integration Tutorial](unreal-engine-integration.md).

## Prerequisites

- Unreal Engine 5.x installed
- GetgudSDK credentials (`titleId` and `privateKey`)
- Basic familiarity with UE5 Blueprints

## Plugin Installation

1. Clone or download the Getgud repository:
   ```
   git clone https://github.com/getgud-io/cpp-getgud-sdk-dev.git
   ```

2. Copy the Blueprint plugin folder to your project's `Plugins/` directory:
   ```
   cpp-getgud-sdk-dev/examples/unreal-engine-5-blueprint/GetgudSDKBlueprint/
   ```

3. Copy SDK header files from `cpp-getgud-sdk-dev/include/` to `Plugins/GetgudSDKBlueprint/ThirdParty/GetgudSDK/include/`:
   ```
   ThirdParty/GetgudSDK/include/
   ├── GetgudSDK.h
   └── actions/
       ├── AffectAction.h
       ├── AttackAction.h
       ├── BaseActionData.h
       ├── DamageAction.h
       ├── DeathAction.h
       ├── HealAction.h
       ├── Helpers.h
       ├── PositionAction.h
       └── SpawnAction.h
   ```

4. Copy the compiled SDK libraries and config:

   **Windows (Win64)** - copy to `Plugins/GetgudSDKBlueprint/ThirdParty/GetgudSDK/bin/Win64/`:
   - `GetgudSDK.dll`
   - `GetgudSDK.lib`
   - `config.json`

   **Linux/Ubuntu** - copy to `Plugins/GetgudSDKBlueprint/ThirdParty/GetgudSDK/bin/Linux/`:
   - `libGetgudSDK.so`
   - `config.json`

   For build instructions, see the [SDK Build Guide](../cpp-build-instructions.md). If you're unable to build the SDK yourself, contact us and we'll build it for your target system.

5. Build your UE5 project

## Configuration

The SDK requires a `config.json` file and creates a log file. There are multiple ways to provide these:

1. **Default behavior**: Call `Init()` - SDK looks for `config.json` next to the executable and creates logs there
2. **Custom path**: Call `InitConfPath(Path)` - provide a specific path to your config file
3. **Inline config**: Call `InitConf(ConfigContent, true)` - pass config JSON as a string
4. **Environment variables** (optional): Set `GETGUD_CONFIG_PATH` and `GETGUD_LOG_FILE_PATH`

**UE5 default paths:**
- Config: `Plugins/GetgudSDKBlueprint/ThirdParty/GetgudSDK/bin/Win64/config.json` (or `Linux/` for Linux)
- Logs: Written to UE5 Engine `Binaries/Win64/` folder (or your platform's equivalent)

## Blueprint Node Reference

All nodes are available under the **Getgud** category in the Blueprint editor.

For detailed parameter descriptions and usage examples, see the [full SDK Commands Reference](../../sdk-commands.md).

### Initialization

| Node | Description |
|------|-------------|
| `Init` | Initialize SDK with default config (looks for config.json next to executable) |
| `InitConfPath` | Initialize SDK with a specific config file path |
| `InitConf` | Initialize SDK with config content or path |
| `Dispose` | Shutdown the SDK |

### Game/Match Management

| Node | Description | Returns |
|------|-------------|---------|
| `StartGame` | Start a new game session | GameGuid (String) |
| `StartMatch` | Start a new match within a game | MatchGuid (String) |
| `MarkEndGame` | Mark a game as ended | Success (Bool) |
| `SetMatchWinTeam` | Set the winning team for a match | Success (Bool) |

**StartGame Parameters:**
- `TitleId` (Int32) - Your Getgud title ID
- `PrivateKey` (String) - Your private API key
- `ServerGuid` (String) - Unique server identifier
- `GameMode` (String) - Game mode name
- `ServerLocation` (String) - Server location

**StartMatch Parameters:**
- `GameGuid` (String) - From StartGame
- `MatchMode` (String) - Match mode name
- `MapName` (String) - Current map
- `CustomField` (String) - Optional custom data

### Actions

| Node | Key Parameters |
|------|----------------|
| `SendSpawnAction` | MatchGuid, PlayerGuid, CharacterGuid, TeamGuid, InitialHealth, Position, Rotation |
| `SendPositionAction` | MatchGuid, PlayerGuid, Position, Rotation |
| `SendAttackAction` | MatchGuid, PlayerGuid, WeaponGuid |
| `SendDamageAction` | MatchGuid, PlayerGuid, VictimPlayerGuid, DamageDone, WeaponGuid |
| `SendDeathAction` | MatchGuid, PlayerGuid, AttackerGuid |
| `SendHealAction` | MatchGuid, PlayerGuid, HealthGained |
| `SendAffectAction` | MatchGuid, PlayerGuid, AffectGuid, AffectState |

All action nodes require:
- `MatchGuid` (String) - From StartMatch
- `ActionTimeEpoch` (Int64) - Unix timestamp in milliseconds

**Position/Rotation Structs:**
- `FGetgudPosition`: X, Y, Z (Float)
- `FGetgudRotation`: Yaw, Pitch, Roll (Float)

### Reports/Chat/Players

| Node | Description |
|------|-------------|
| `SendInMatchReport` | Send a report during a match |
| `SendChatMessage` | Send a chat message (MatchGuid, MessageTimeEpoch, PlayerGuid, Message) |
| `SendReport` | Send an out-of-match report with credentials |
| `UpdatePlayer` | Update player information with credentials |

## Basic Integration Workflow

```
1. BeginPlay → Init()

2. On Game Start:
   StartGame(TitleId, PrivateKey, ServerGuid, GameMode, Location)
   → Save returned GameGuid

3. On Match Start:
   StartMatch(GameGuid, MatchMode, MapName, CustomField)
   → Save returned MatchGuid

4. During Gameplay:
   - Player spawns → SendSpawnAction(...)
   - Player moves → SendPositionAction(...) [every tick or at intervals]
   - Player attacks → SendAttackAction(...)
   - Player damages → SendDamageAction(...)
   - Player dies → SendDeathAction(...)
   - Player heals → SendHealAction(...)
   - Buff/debuff → SendAffectAction(...)
   - Chat message → SendChatMessage(...)

5. On Game End:
   MarkEndGame(GameGuid)

6. On Shutdown:
   Dispose()
```
