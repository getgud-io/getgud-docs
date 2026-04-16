<div align="center">
  <img style="width: 512px" src="https://getgud-public-content.s3.amazonaws.com/gg-cover.png">
</div>

# Getgud.io Football Game Mapping Guide

This document describes how to correctly model Players, NPCs, Ball, Teams, Affects, and Gameplay Signals to achieve full analytics coverage in football (soccer) games.

Getgud.io fully supports football-style games with:

- 11 vs 11 player matches  
- Dynamic player control switching  
- AI/NPC-controlled teammates and opponents  
- A shared ball entity  
- Continuous gameplay systems (stamina, power, pressure)  
- Real-time positional gameplay  

---

## Player Mapping

Every footballer must include:

- `player_guid`  
- `character_guid`  
- `team_guid`  

### Player GUID

When a user controls a footballer, `player_guid` must be a stable, unique identifier for the user.

```
1125555
```

### Character GUID

Represents the footballer identity.

Examples:

- Messi  
- Ronaldo  
- Mbappe  
- Bellingham  

### Player Team GUID

Represents the team the footballer belongs to.

Examples:

- Spain  
- Germany  
- Blue-Team  
- Home-Team  

### Example

```cpp
GetgudSDK::SendAction(new GetgudSDK::SpawnActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "Messi",
  "Spain",
  100.f,
  GetgudSDK::PositionF{12.0f, 8.0f, 0.0f},
  GetgudSDK::RotationF{0.0f, 0.0f, 0.0f}
));
```

---

## NPC Mapping

All footballers not controlled by a user are NPCs.

Each NPC must have:

- `player_guid` = `NPC_<footballer name>`  
- `character_guid` = `NPC_<footballer name>`  
- `team_guid` = team name  

### NPC GUID

Prefix of all NPCs must be `NPC_`.

Examples:

```
NPC_Messi
NPC_Ronaldo
NPC_Mbappe
NPC_Defender_1
```

### Example

```cpp
GetgudSDK::SendAction(new GetgudSDK::SpawnActionData(
  matchGuid,
  curTimeEpoch,
  "NPC_Messi",
  "NPC_Messi",
  "Spain",
  100.f,
  GetgudSDK::PositionF{10.0f, 5.0f, 0.0f},
  GetgudSDK::RotationF{0.0f, 0.0f, 0.0f}
));
```

---

## Player Switching (Core Football Mechanic)

A user controls only one footballer at a time but can switch control during the match.

Whenever a user switches control from one footballer to another, you must send two Spawn actions.

### Example: User switches from Ronaldo → Messi

#### 1. Spawn new user-controlled footballer

```cpp
GetgudSDK::SendAction(new GetgudSDK::SpawnActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "Messi",
  "Spain",
  100.f,
  GetgudSDK::PositionF{12.0f, 8.0f, 0.0f},
  GetgudSDK::RotationF{0.0f, 0.0f, 0.0f}
));
```

#### 2. Spawn previous footballer as NPC

```cpp
GetgudSDK::SendAction(new GetgudSDK::SpawnActionData(
  matchGuid,
  curTimeEpoch,
  "NPC_Ronaldo",
  "NPC_Ronaldo",
  "Spain",
  100.f,
  GetgudSDK::PositionF{5.0f, 3.0f, 0.0f},
  GetgudSDK::RotationF{0.0f, 0.0f, 0.0f}
));
```

---

## Ball Mapping

The ball is treated as a PvE (environment) entity.

### Ball Rules

- `player_guid` = `PvE_ball`  
- `character_guid` = `ball`  
- `team_guid` = `ball`  

### Ball Spawn

```cpp
GetgudSDK::SendAction(new GetgudSDK::SpawnActionData(
  matchGuid,
  curTimeEpoch,
  "PvE_ball",
  "ball",
  "ball",
  0.f,
  GetgudSDK::PositionF{0.0f, 0.0f, 0.0f},
  GetgudSDK::RotationF{0.0f, 0.0f, 0.0f}
));
```

---

## Affects

Affects represent everything happening to a footballer.

Football games usually need only the Affect states:

- `Activate`  
- `Deactivate`

and not need:

- `Attach`  
- `Detach`  

This is because football actions are generally momentary or state-based, rather than inventory-based.

---

## Types of Affects

Most football games have mechanics in the following Affect categories.

### Movement Affects

- `sprint`  
- `jog`  
- `turn`  
- `stop`  

### Ball Interaction Affects

- `pass`  
- `short_pass`  
- `long_pass`  
- `through_pass`  
- `cross`  
- `low_cross`  
- `shot`  
- `header`  
- `first_touch`  

### Dribbling Affects

- `dribble`  
- `skill_move`  
- `ball_control`  

### Defensive Affects

- `pressure`  
- `tackle_standing`  
- `tackle_slide`  
- `interception`  
- `block`  

### Goalkeeper Affects

- `goalkeeper_save`  
- `goalkeeper_dive`  
- `goalkeeper_catch`  
- `goalkeeper_punch`  

### Physical Affects

- `jump`  
- `collision`  
- `fall`  

---

## Affect Examples

### Sprint Start

```cpp
GetgudSDK::SendAction(new GetgudSDK::AffectActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "sprint",
  GetgudSDK::AffectState::Activate
));
```

### Sprint Stop

```cpp
GetgudSDK::SendAction(new GetgudSDK::AffectActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "sprint",
  GetgudSDK::AffectState::Deactivate
));
```

### Pressure Start

```cpp
GetgudSDK::SendAction(new GetgudSDK::AffectActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "pressure",
  GetgudSDK::AffectState::Activate
));
```

### Slide Tackle

```cpp
GetgudSDK::SendAction(new GetgudSDK::AffectActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "tackle_slide",
  GetgudSDK::AffectState::Activate
));
```

### Header

```cpp
GetgudSDK::SendAction(new GetgudSDK::AffectActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "header",
  GetgudSDK::AffectState::Activate
));
```

---

## Affects with Scale

Football games often have continuous gameplay values that should also be modeled through Affects.

Examples include:

- stamina  
- shot power  
- pass power  
- pressure intensity  
- dribble intensity  

### Format

Use the following naming convention:

```text
<affect>-Scale-<value>
```

### Examples

- `stamina-Scale-73`  
- `shot_power-Scale-91`  
- `pressure-Scale-65`  
- `pass_power-Scale-40`  

### Example: Stamina Reporting

```cpp
GetgudSDK::SendAction(new GetgudSDK::AffectActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "stamina-Scale-73",
  GetgudSDK::AffectState::Activate
));
```

### Example: Shot Power Reporting

```cpp
GetgudSDK::SendAction(new GetgudSDK::AffectActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "shot_power-Scale-91",
  GetgudSDK::AffectState::Activate
));
```

### Scale Guidelines

- Scale values should typically be between `0` and `100`  
- Scale values can be sent periodically, for example every 1 to 10 seconds  
- Each scale Affect must be tied to the relevant player  

---

## Winning Team Reporting

At the end of the football match, the winning team should be reported.

### Example

```cpp
GetgudSDK::SetMatchWinTeam(matchGuid, "Spain");
```

This allows Getgud.io to analyze match outcome by team.

---

## Summary Checklist

### User-Controlled Footballers

- `player_guid` = real user GUID  
- `character_guid` = actual footballer identity  
- `team_guid` = actual team name  

### NPC-Controlled Footballers

- `player_guid` = `NPC_<footballer identity>`  
- `character_guid` = `NPC_<footballer identity>`  
- `team_guid` = actual team name  

### User Switching

- Every control switch must send two Spawn actions  
- New footballer becomes user-controlled  
- Previous footballer becomes NPC-controlled  

### Ball

- `player_guid` = `PVE_ball`  
- `character_guid` = `ball`  
- `team_guid` = `ball`  

### Affects

- Use football-specific Affects  
- Use primarily `Activate` and `Deactivate`  

### Affects with Scale

- Use `<affect>-Scale-<value>`  
- Use for stamina, power, pressure, and other continuous values  

### Position Tracking

- User-controlled footballers: `10-32 Hz`  
- NPC footballers: `1-5 Hz`  
- Ball: `10-32 Hz`  

---

## Final Notes

Following this design ensures:

- accurate football replay reconstruction  
- clear distinction between user-controlled and NPC-controlled footballers  
- correct ball visualization  
- meaningful football-specific analytics  
- scalable support across different football games  


