<div align="center">
  <img style="width: 512px" src="https://getgud-public-content.s3.amazonaws.com/gg-cover.png">
</div>

# Getgud.io Open-World Mapping Guide

This document describes how to correctly model Players, NPCs, Weapons, Maps, Affects, and Zones to achieve full analytics coverage in open-world and RPG-style games.

Getgud.io fully supports large-scale, open-world, and single-player games similar to Final Fantasy, Dragon’s Dogma, Skyrim, and other RPGs with:

- Thousands of NPCs  
- Dozens of NPC types  
- Massive open-world maps  
- Player character classes  
- Weapons with buff permutations  
- Single-player or multiplayer scenarios  

---

## Player Mapping

Every player must include:

- `player_guid`  
- `character_guid`  
- `team_guid`  

### Player GUID

A stable, unique identifier for the player.

```
347589734578
```

### Character GUID

Represents the class, archetype, or form of the player.

Examples:

- Wizard  
- Fighter  
- Rogue  
- Paladin  

### Player Team GUID

Required even in single-player.

Examples:

- Player-Team  
- Hero-Team  
- MainTeam  

### Example

```yaml
player_guid: 347589734578
character_guid: Wizard
team_guid: Player-Team
```

---

## NPC Mapping

NPCs in open-world RPGs often spawn in large numbers. Each NPC must have:

- `npc_guid` (unique per NPC instance alive at the same time)  
- `character_guid` (type/archetype)  
- `team_guid` (faction/group)  

### NPC GUID (instance identifier)

Best practice: every NPC GUID should start with `NPC_`.

Examples:

```
NPC_small_zombie_1
NPC_small_zombie_2
NPC_small_zombie_100
NPC_big_zombie_1
NPC_big_zombie_2
NPC_orc_boss_1
NPC_orc_boss_2
```

If you want an NPC to appear in the replay and 3D modeler, but **not** be analyzed in any Getgud.io metric (no insights, no behavior tracking, no scoring), prefix the GUID with `PvE_`:

```
PvE_small_zombie_1
PvE_forest_deer_12
```

### NPC Character GUID (type/class)

Represents the archetype, not the specific instance.

Examples:

- small_zombie  
- big_zombie  
- orc_boss  
- forest_dragon  

All small zombies share the same `character_guid`, even though their `npc_guid`s differ.

### NPC Team GUID (faction)

Assign each NPC type to the appropriate team:

- zombie_team  
- orc_team  
- wildlife_team  
- boss_team  

This enables Getgud.io to group behaviors, interactions, and combat analytics by faction.

---

## Winning Team Reporting

At the end of a boss fight, battle, encounter, or zone-level combat session, you should call:

```
sendMatchWinningTeam(<team_guid>)
```

If the player wins:

```
sendMatchWinningTeam("Player-Team")
```

If NPCs win (player dies, wipes, fails event):

```
sendMatchWinningTeam("zombie_team")
```

Reporting the winning team allows Getgud.io to:

- Track success/fail rates  
- Understand difficulty curves  
- Analyze player skill progression  
- Identify frustrating encounters  

---

## Weapon Mapping

Weapons should include their type and any buffs, enchantments, temporary effects, or permutations in a single identifier.

Base example:

```
two_handed_sword
```

With poison buff:

```
two_handed_sword_poison
```

With fire buff:

```
two_handed_sword_fire
```

With multiple buffs:

```
two_handed_sword_fire_poison
```

Each permutation can have different:

- Damage profiles  
- Status effects  
- Combat performance  
- Player behavior impact  

Getgud.io analyzes each weapon permutation separately.

---

## Respawns and Reuse of NPC GUIDs

### Best practice

When an NPC dies and respawns in the same zone, reuse the same NPC GUID:

```
NPC_small_zombie_14  (dies)
NPC_small_zombie_14  (respawns)
```

### Avoid

Creating new GUIDs on every respawn:

```
NPC_small_zombie_101
NPC_small_zombie_102
```

This inflates analytics and breaks continuity for that NPC type.

---

## Map and Zone Mapping

Map GUID = map name. Use meaningful, readable names.

Examples:

```
burning_village
eastern_forest
ancient_ruins
mountain_pass
crystal_caverns
```

Each map/zone must be treated as a separate match session.

---

## Match Boundaries in Open Worlds

A “match” = time spent in one map/zone.

When the player transitions from:

```
burning_village → eastern_forest
```

You should:

1. End the current match  
2. Start a new match  
3. Optionally upload the 3D map for visualization  

This keeps the analytics clean, efficient, and meaningful.

---

## NPC Counting Rules Across Zones

You have two options:

### Option A: Reset NPC numbering per map

Zone 1:

```
NPC_small_zombie_1 … NPC_small_zombie_100
```

Zone 2:

```
NPC_small_zombie_1 … NPC_small_zombie_80
```

This is simpler and perfectly acceptable.

### Option B: Global NPC numbering across the entire world

Zone 1:

```
NPC_small_zombie_1 … NPC_small_zombie_100
```

Zone 2:

```
NPC_small_zombie_101 … NPC_small_zombie_181
```

Use this if you want per-creature lifetime analytics across the entire world.

---

## Tickrate Recommendations

### NPCs

NPC movement is usually not critical for precision analytics. Send 1 position update per second:

```
NPC position: 1 Hz
```

This prevents match file bloating.

### Player

Recommended send rates:

- Single-player: `5–10 Hz`  
- Multiplayer / future-proof anti-cheat: `32 Hz` minimum  

---

## Affects

Affects are one of the most powerful features of the Getgud.io platform.

They allow you to represent any game-specific mechanic (abilities, buffs, debuffs, consumables, spells, items, status effects, etc.) in a game-agnostic, structured, and analyzable way.

Affect events describe what is happening to a player or NPC, when it happens, and why.

Affect GUIDs are defined by your game logic, for example:

- `health_potion`  
- `rage_buff`  
- `fire_enchant`  
- `combo_strike`  

Each affect has four possible states, representing the full lifecycle of an item, ability, buff, or mechanic.

### Affect States

The four affect states are:

1. ATTACH  
2. DETACH  
3. ACTIVATE  
4. DEACTIVATE (optional)  

#### ATTACH

Indicates the moment an affect is gained, acquired, or applied to a player or NPC.

Examples:

- Player picks up a health potion → `health_potion`, state: `attach`  
- Player receives a poison debuff from stepping in toxic gas → `poison_debuff`, state: `attach`  
- Player learns or equips a new spell → `fireball_spell`, state: `attach`  

Key rule: ATTACH is inventory/availability level, not activation level.

#### DETACH

Indicates the moment the affect is lost, removed, or no longer available, no matter the cause.

Examples:

- Potion was consumed → `health_potion`, state: `detach`  
- Buff expired naturally → `poison_debuff`, state: `detach`  
- Spell book discarded → `fireball_spell`, state: `detach`  
- Weapon unequipped → `two_handed_sword`, state: `detach`  

#### ACTIVATE

Indicates the moment the affect is used, triggered, or activated.

Examples:

- Player drinks the health potion → `health_potion`, state: `activate`  
- Player casts fireball → `fireball_spell`, state: `activate`  
- Player enters rage mode → `rage_buff`, state: `activate`  
- Player triggers a combo strike → `combo_strike`, state: `activate`  

Most gameplay-specific insights come from ACTIVATE events.

#### DEACTIVATE (optional)

Indicates the moment an affect’s in-game effect ends.

Examples:

- Rage mode expires → `rage_buff`, state: `deactivate`  
- Fly spell duration ends → `fly_spell`, state: `deactivate`  

You only need to send this if it matters to gameplay or analysis.

### Why Affects Matter

Affects allow Getgud.io to track:

- Consumable usage  
- Spell activations  
- Damage-over-time effects  
- Temporary buffs and debuffs  
- Combo chains  
- Class abilities  
- Weapon enchantments  
- Movement boosts  
- Crowd control effects  
- Status ailments  
- Mechanics unique to your game  

Affects create a rich behavioral timeline, enabling:

- Insight generation  
- Balance analysis  
- Player behavior pattern analysis  
- Build/gear effectiveness studies  
- Ability performance tracking  
- Usage frequency metrics  
- Combat sequence reconstruction  
- Anti-cheat anomaly detection  

In simple terms:

> Affects = your game’s vocabulary expressed in Getgud.io format.

### Naming and Structure

Affect GUIDs can be anything meaningful, clear, and reusable across the game.

Examples:

```
fireball
rage_buff
health_potion
slow_debuff
two_handed_sword_poisoned
berserk_combo_stage2
```

Structure recommendation:

- `<category>_<effect>`  
- `<item>_<modifier>`  
- `<spell>_<tier>`  

You can choose any system as long as it’s consistent.

---

## Summary Checklist

### Player

- Unique `player_guid`  
- Player `character_guid`  
- Player `team_guid`  

### NPCs

- GUID starting with `NPC_`  
- Unique per instance on the map  
- Character GUID = NPC type  
- Team GUID = NPC faction  
- Reuse GUIDs for respawns  
- Use `PvE_` prefix to ignore NPC analysis  

### Weapons

- Include type + buff(s)  
- Example: `magic_staff_fire`  

### Map/Zone

- Map GUID = map name  
- One match per zone  
- End match when changing zones  

### Tickrates

- NPCs: 1/sec  
- Player: 10–32/sec  

### Winning Team

- Use SDK method `sendMatchWinningTeam(team_guid)` at the end of any encounter  

### Affects

- Use affect states to model abilities, buffs, spells, consumables, items, and mechanics  

---

## Final Notes

Following this design ensures:

- Accurate replay reconstruction  
- Clean analytics across massive worlds  
- Efficient data usage  
- Clear team-based win/loss insights  
- Per-type and per-permutation weapon analysis  
- Stable long-term NPC tracking  
- Fully visualized maps and combat zones  

Your open-world game will be completely analyzable inside Getgud.io, with powerful insights into player performance, difficulty balance, encounter design, weapon effectiveness, and NPC behavior.
