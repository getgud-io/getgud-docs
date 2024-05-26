# Get started with Getgud.io

This guide will walk you through the initial steps to get started with Getgud.io, a platform that provides observability & insights on your game & players. With Getgud you can know and act on everything your players are doing, including if they cheat or grief!

For better understanding we recommend watching our [onboarding tutorial](https://www.youtube.com/watch?v=4a7rFfUTUrI)

## Before you begin

To get started, create an account with Getgud.io.

- [Sign up](https://staging.dashboard.getgud.io/auth/register/)
  - If you've never used Getgud.io before, sign up for a new Getgud account
 
- [Log in](https://staging.dashboard.getgud.io/auth/login/)
  - If you already have a Getgud account, log in to get started
 
Once you create an account, you can create your first title with Getgud.

## Create your first title

To create your first title visit [Manage Titles](https://staging.dashboard.getgud.io/manage/titles/) section on the dashboard. 

- On the right upper corner of the screen find `Title Actions` button, press the button, and then click `Add Title`.
- In the form pop-up up fill your title name, title franchise and optionally add a logo.
- Title name and title franchise may be the same name! Example: Title Name: `Apex Legends`, Title Franchise `Apex Legends`
- But they also can be different names. Example: Title Name: `CS:GO`, Title Franchise `Counter Strike`

## Invite your team





## Prerequisites

To start, we need to understand the basic structure GetGud uses to describe a video game: 

**Titles->1->N->Games->1->N->Matches->1->N->Actions**

* The top container in Getgud is `Title`, which represents a literal game’s title, you as a client can have many titles, for example, a `Title` named CS:GO represents the CS:GO video game.

  ```
  An example of a Title: CS:GO 
  ```

* Next up is `Game`, a `Game` is a container of matches that belong to the same `Title` from the same server session, where mostly the same players in the same teams, play one or more `Matches` together. You as a client can identify every game with a unique `gameGuid` that is provided to you once the `Game` starts. 

  ```
  An example of a Game is a CS:GO game which has 30 matches (also known as rounds) within it.
  ```

* `Match` represents the actual play time that is streamed for analysis.
A `Match` contains the actions that occurred during the match's timespan.
Like `Game`, `Match` also has a GUID which will be provided to you once you start a new match.

  ```
  An example of a Match is a single CS:GO round inside the game.
  ```

* `Action` represents an in-match activity that is associated with a player. We collect seven different action types which are common to all first person shooter gamnes:
1. `Spawn` - Whenever a player appears or reappears in-match, on the map.
2. `Death` - A death of a player, either by another player, the environemnt or the player itself.
3. `Position` - player position change (including looking direction).
4. `Attack` - Whenever a player initiates any action that might cause damage, now or in the future. Examples: shooting, throwning a granade, planting a bomb, swinging a sword, punching, firing a photon torpedo, etc.
5. `Damage` - Whenever a player recieves any damage, either from another player, the environemnt or the player itself.
6. `Heal` - Whenever a player is healed, doesn't matter by whom or how.
7. `Affect` - Whenever an in-match affect of any kind is applied to the player. Examples: crouch, prone, jump, fly, use special ability, boost speed/ammo/shield/health, etc. 
