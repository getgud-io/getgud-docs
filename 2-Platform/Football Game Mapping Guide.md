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

- `player_guid` = `NPC_<character>`  
- `character_guid` = `NPC_<character>`  
- `team_guid` = team  

### NPC GUID

Best practice: prefix all NPCs with `NPC_`.

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

- `player_guid` = `PVE_ball`  
- `character_guid` = `ball`  
- `team_guid` = `ball`  

### Ball Spawn

```cpp
GetgudSDK::SendAction(new GetgudSDK::SpawnActionData(
  matchGuid,
  curTimeEpoch,
  "PVE_ball",
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

Football games typically use:

- ACTIVATE  
- DEACTIVATE  

### Example

```cpp
GetgudSDK::SendAction(new GetgudSDK::AffectActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "sprint",
  GetgudSDK::AffectState::Activate
));
```

---

## Affect Scale

Use:

```
<affect>-Scale-<value>
```

Example:

```cpp
GetgudSDK::SendAction(new GetgudSDK::AffectActionData(
  matchGuid,
  curTimeEpoch,
  "1125555",
  "stamina-Scale-73",
  GetgudSDK::AffectState::Activate
));
```

---

## Summary Checklist

- User → real GUID  
- NPC → NPC_<character>  
- Ball → PVE_ball  
- 2 spawn events per switch  
- Use Affects for gameplay  
- Use Scale for continuous values  

