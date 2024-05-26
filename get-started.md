# Get Started With Getgud.io

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