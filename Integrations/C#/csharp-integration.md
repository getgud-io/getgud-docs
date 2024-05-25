# GetGud C# SDK

## What Can You Do With GetGud's SDK:

GetGud's C# SDK allows you to integrate your game with the GetGud platform. Once integrated, you will be able to:

- Stream live game data to GetGud's cloud, including in-match actions, in-match reports, and in-match chat messages.
- Send reports about historical matches to GetGud.
- Send (and update) player information to GetGud.

## Build Requirements:
The C# SDK requires the following build requirements:

- Visual Studio or any other C# development environment.
- .NET Framework or .NET Core installed.

## Getting Started

To get started with the GetGud C# SDK, follow the steps below:

### Step 1: Install the GetGud SDK Package

1. Create a new C# project in your preferred development environment.
2. Open the NuGet Package Manager or the Package Manager Console.
3. Search for the "GetGudSdk" package and install it in your project.

### Step 2: Initialize the SDK

To initialize the SDK, import the GetGudSdk namespace and call the `Init()` method:

```csharp
using GetGudSdk;

class Program
{
    static void Main(string[] args)
    {
        // Initialize the SDK
        int status = GetGudSdk.Methods.Init();

        // Handle the initialization status
        if (status == 0)
        {
            Console.WriteLine("SDK initialized successfully.");
        }
        else
        {
            Console.WriteLine("Failed to initialize the SDK.");
        }

        // Continue with other SDK operations

        // Dispose of the SDK when finished
        GetGudSdk.Methods.Dispose();
    }
}
```

### Step 3: Start a Game

To start a new game, call the `StartGame()` method:

```csharp
// Initialize input and output data for a Game
string gameGuid;

GetGudSdk.StartGameInfo gameInfo = new GetGudSdk.StartGameInfo
{
    ServerGuid = "123",
    PrivateKey = "8526b010-0c56-11ee-bc1d-9f343a78df6b",
    TitleId = 30,
    GameMode = "gameMode"
};

// Start a Game
int status = GetGudSdk.Methods.StartGame(gameInfo, out gameGuid);

// Handle the game start status and gameGuid
if (status == 0)
{
    Console.WriteLine("Game started successfully. Game Guid: " + gameGuid);
}
else
{
    Console.WriteLine("Failed to start the game.");
}
```

### Step 4: Start a Match

Once you have started a game, you can start a match within that game by calling the `StartMatch()` method:

```csharp
// Initialize input and output data for a Match
string matchGuid;

GetGudSdk.StartMatchInfo matchInfo = new GetGudSdk.StartMatchInfo
{
    GameGuid = gameGuid,
    MapName = "map",
    MatchMode = "matchMode"
};

// Start a Match of the Game
status = GetGudSdk.Methods.StartMatch(matchInfo, out matchGuid);

// Handle the match start status and matchGuid
if (status == 0)
{
    Console.WriteLine("Match started successfully. Match Guid: " + matchGuid);
}
else
{
    Console.WriteLine("Failed to start the match.");
}
```

### Step 5: Send Actions to the Match

Once you have a match, you can send various types of actions to it, such as spawn actions, position actions, attack actions, etc. Here's an example of sending a spawn action:

```csharp
// Initialize input data for a Spawn Action
GetGudSdk.SendSpawnActionInfo spawnInfo = new GetGudSdk.SendSpawnActionInfo
{
    BaseData = new GetGudSdk.BaseActionData
    {
        MatchGuid = matchGuid,
        PlayerGuid = "player-1",
        ActionTimeEpoch = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds()
    },
    CharacterGuid = "LL",
    InitialHealth = 100,
    TeamId = 1,
    Position = new GetGudSdk.PositionF { X = 10F, Y = 10F, Z = 0.0001F },
    Rotation = new GetGudSdk.RotationF { Roll = 240F, Pitch = 180F, Yaw = 0F }
};

// Send the Spawn Action to the SDK
GetGudSdk.Methods.SendSpawnAction(spawnInfo);
```
You can similarly send other types of actions using the respective methods provided by the SDK.

### Step 6: End the Game

When a game is finished, you should mark it as ended to close it on the GetGud platform. To do this, call the `MarkEndGame()` method:

```csharp
// End the Game
status = GetGudSdk.Methods.MarkEndGame(gameGuid);

// Handle the game end status
if (status == 0)
{
    Console.WriteLine("Game ended successfully.");
}
else
{
    Console.WriteLine("Failed to end the game.");
}
```

### Step 7: Dispose of the SDK

Once you have finished working with the SDK, make sure to dispose of it by calling the `Dispose()` method:

```csharp
GetGudSdk.Methods.Dispose();
```
This will release any resources used by the SDK.

## Additional SDK Methods

The GetGud C# SDK provides several other methods for sending chat messages, reports, updating player information, and more. Refer to the SDK documentation and examples for detailed usage instructions of these methods.

## Configuration

The SDK allows you to configure various settings through a configuration file named `config.json`. The default configuration file is automatically loaded during SDK initialization. You can modify the configuration file as per your requirements. See the SDK documentation for more details on the available configuration options.

## Examples

The GetGud C# SDK repository provides examples that demonstrate the usage of the SDK in various scenarios. You can refer to these examples to understand how to integrate the SDK into your own C# projects.

For more information and detailed usage instructions, refer to the GetGud C# SDK documentation and the provided examples.
