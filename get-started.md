# Get started with Getgud.io

This guide will walk you through the initial steps to get started with Getgud.io, a platform that provides observability and insights on your game and players. With Getgud, you can monitor and act on everything your players are doing, including detecting cheating or griefing.

For a better understanding, we recommend watching our [Onboarding tutorial](https://www.youtube.com/watch?v=4a7rFfUTUrI).

## Before you begin

To get started, create an account with Getgud.io.

- [Sign up](https://staging.dashboard.getgud.io/auth/register/)
  - If you or your organization has never used Getgud.io before, sign up for a new Getgud account.
 
- [Log in](https://staging.dashboard.getgud.io/auth/login/)
  - If you or your organization already has a Getgud account, log in to get started.
  - Note: If you are a developer, you may need to ask your organization admin to send you an invite to the Getgud.io platform.

Once you create an account, you can create your first title with Getgud.

## Create your first title

To create your first title, visit the [Manage Titles](https://staging.dashboard.getgud.io/manage/titles/) section on the dashboard.

- In the upper right corner of the screen, find the `Title Actions` button, press it, and then click `Add Title`.
- In the form pop-up, fill in your Title Name, Title Franchise, and optionally add a logo.
  - Title Name and Title Franchise may be the same. Example: Title Name: `Apex Legends`, Title Franchise: `Apex Legends`
  - They can also be different. Example: Title Name: `CS:GO`, Title Franchise: `Counter-Strike`

## Invite your team

Once you create an account, you can start adding members of your organization to Getgud.

- Visit the [Users](https://staging.dashboard.getgud.io/settings/users/) page in settings.
- In the upper right corner of the screen, find the `Actions` button, press it, and then click `Invite User`.
- In the form pop-up, fill in the First Name, Last Name, Email, and pick the User Role, then click `Invite`.
- The invited user will receive further instructions via email.
- Note: You can manage user roles and permissions by clicking the `Actions` button and then pressing `Manage User Roles`.

## What Can You Do With Getgud's SDK:

Getgud's SDK allows you to integrate your game with our platform. Once integrated, you will be able to:

1. Stream live game data to GetGud's cloud, including in-match actions, in-match reports, and in-match chat messages.
2. Send Reports about historical matches to Getgud.
3. Send (and update) player information to Getgud.

## What Data Does The SDK Collect 

Now, we need to understand the basic structure GetGud uses to describe a video game:

**Titles -> 1 -> N -> Games -> 1 -> N -> Matches -> 1 -> N -> Actions**

- The top container in Getgud is `Title`, which represents a literal game’s title. As a client, you can have many titles. For example, a `Title` named CS:GO represents the CS:GO video game.

  ```
  Example of a Title: CS:GO
  ```

- Next is `Game`, a container of matches that belong to the same `Title` from the same server session, where mostly the same players in the same teams play one or more `Matches` together. Each game can be identified with a unique `gameGuid` provided once the `Game` starts.

  ```
  Example of a Game: A CS:GO game with 30 matches (also known as rounds).
  ```

- `Match` represents the actual playtime that is streamed for analysis. A `Match` contains the actions that occurred during its timespan. Like `Game`, `Match` also has a GUID provided once you start a new match.

  ```
  Example of a Match: A single CS:GO round inside the game.
  ```

- `Action` represents an in-match activity associated with a player. We collect seven different action types common to all first-person shooter games:
  1. `Spawn` - Whenever a player appears or reappears in-match, on the map.
  2. `Death` - A death of a player, either by another player, the environment, or the player themselves.
  3. `Position` - Player position change (including looking direction). This action should be sent every X Ticks (minimum of 32 tick rate).
  4. `Attack` - Whenever a player initiates any action that might cause damage, now or in the future. Examples: shooting, throwing a grenade, planting a bomb, swinging a sword, punching, firing a photon torpedo, etc.
  5. `Damage` - Whenever a player receives any damage, either from another player, the environment, or the player themselves.
  6. `Heal` - Whenever a player is healed, no matter by whom or how.
  7. `Affect` - Whenever an in-match effect of any kind is applied to the player. Examples: crouch, prone, jump, fly, use special ability, boost speed/ammo/shield/health, etc.

## How to start the integration process

Getgud is integrated **server-side** into your game. Our SDK is built in C++, and we provide both source code and build files for popular systems like Windows and Linux. We offer header files in the most popular languages so you can integrate our code with C++, C#, C, and Python servers. If you are using Unreal Engine 4, Unreal Engine 5, or Unity, we provide separate instructions for integrating with those engines.

To get started:

1. Decide if you are going to build the SDK from source or use our build files.
2. If you are building from scratch, clone our [development repo](https://github.com/getgud-io/getgud-docs/blob/main/Integrations/cpp-build-instructions.md) and follow the [build instructions](https://github.com/getgud-io/cpp-getgud-sdk-dev).
3. Use the build files and appropriate header files to start integrating into your server.
4. Ensure you understand the Getgud SDK [lifecycle and events](https://github.com/getgud-io/getgud-docs/blob/main/Integrations/C%2B%2B/cpp-integration.md) you can send.
