# SDK Integration FAQ

This page answers common questions about integrating Getgud's SDK actions into your game. It covers best practices for all 7 action types: **Spawn**, **Position**, **Attack**, **Affect**, **Damage**, **Heal**, and **Death**.

For the full SDK API reference, see [SDK Commands](https://github.com/getgud-io/getgud-docs/blob/main/sdk-commands.md).

---

## Spawn

**Should Spawn be triggered only once at initial spawn, even if the character is teleported right after?**

Yes, one Spawn per life cycle. If there's a small delay between the actual game spawn and when you call Spawn, that's fine — call it whenever works best for your game logic. If a teleport happens right after, handle that separately with an Affect.

**When should teleportation be treated as a Spawn vs a position update?**

Teleportation should never be a Spawn — the player didn't die or change character, so there's no new life cycle to start. Always pair teleportation with an Affect instead (see Position section below).

**If a character respawns as a different class or entity, should this be a new Spawn?**

Yes. Spawn carries `characterGuid` and `teamGuid`, which define who the player is in the match. If either changes, send a new Spawn so we can track per-character analytics (movement speed profiles, weapon usage, etc.).

**If a character returns to the original type later, should this trigger a Spawn?**

Only through a death/respawn cycle — send Death, then Spawn with the original `characterGuid`. Mid-life class switches without dying are better tracked via Affects.

**Should reconnecting players trigger a Spawn?**

It depends. If your game kills the player on disconnect, send Death — then Spawn when they reconnect (since they need to respawn). If the player stays alive while disconnected, use an Affect (`"disconnected", Activate` → `"disconnected", Deactivate` on reconnect). The logic: Spawn means "a new life begins." If they didn't die, there's no new life.

---

## Position

**Should position be sent every tick, or is there a threshold-based approach?**

We recommend a minimum of **32 position updates per second**. You can send more if your server runs at a higher tick rate — the SDK delta-compresses positions, so bandwidth scales well. Consistency matters: steady position data gives better analytics than sporadic updates.

**How should teleportation be distinguished from normal movement in position updates?**

Always pair teleportation with an Affect (`"teleport", Activate` → position update → `"teleport", Deactivate`). The position update itself looks the same as any other — just new coordinates. The Affect provides the context that the movement is intentional and not anomalous. This applies to both player-initiated teleports (abilities) and game-initiated teleports (server moving a player).

**Should all sudden position changes be paired with an Affect?**

Yes. Any movement that isn't normal walking/running/falling should have an accompanying Affect. If a movement looks unnatural, our systems will flag it unless an Affect provides context.

**Should the Affect be sent before or after the position update?**

Before. The sequence should be: `Affect Activate` → position update(s) → `Affect Deactivate`. We process actions chronologically, so we need to know the ability is active before we see the movement.


**Should velocity or movement state be included in position events?**

Position only carries X, Y, Z and Yaw, Pitch, Roll — no velocity or state fields. Movement states like sprinting or crouching should be sent as Affects (e.g., `"sprint", Activate` / `"sprint", Deactivate`).

**How should out-of-bounds positions be handled?**

Send the actual server position as-is. Don't filter or clamp positions.

---

## Attack

**Should Attack events be sent even if no damage is applied?**

Yes. Attack means "the player attempted to cause damage." It doesn't matter if the attack hits or misses.

**For spawned objects (traps, deployables), should Attack be triggered on deployment or on damage?**

On deployment — that's when the player performed the action. The Attack is the placement, not the trigger. When the object later triggers and deals damage, send a Damage event at that time, attributed to the player who placed it with the appropriate `weaponGuid`.

**For delayed or conditional abilities (parry, counter, etc.), when should Attack be sent?**

At activation. Attack represents the player's intent to cause damage, not the outcome.

**For multi-stage abilities, should each stage trigger its own Attack?**

Yes, if each stage is a separate attempt to deal damage. A 3-round burst = 3 Attacks. A channeled beam = Attack at the start, then individual Damage events as they tick. The key question: "is this a separate attempt to hit something?" If yes, it's a separate Attack.

**Should each projectile instance generate a separate Attack?**

Yes. Each projectile is a distinct player action with its own timestamp. Two projectiles = two Attacks with the same `weaponGuid`.

**How should friendly fire or self-damage be handled?**

Send Attack and Damage events as normal. The attacker and victim can be the same player (self-damage) or teammates. Don't filter these out — we analyze friendly fire patterns as part of our detection.

**Should spawning an NPC be considered an Attack?**

No. Spawning a summon/NPC is an ability use — track it as an Affect (`"summon", Activate` when summoned, `"summon", Deactivate` when it despawns).

---

## Affect

**Should equippable items use Activate/Deactivate or Attach/Detach?**

Use whatever maps naturally to your game logic. The four states serve different purposes:
- **Attach/Detach** = the player has or doesn't have something (inventory/loadout)
- **Activate/Deactivate** = something is actively affecting the player right now

For a weapon pickup: Attach on pickup, Detach on drop. For a consumable: Activate when used, Deactivate when the effect wears off. You don't need all four — just the ones that make sense.

**Are all four Affect states required?**

No. We may use all 4 states for gameplay tracking, or just some — it depends on the context. Send them in whatever way maps best to your game logic.

**For debuffs that deal damage over time, should we send both Attack and Affect?**

Yes — they serve different purposes. The Affect tracks the debuff state (Activate when applied, Deactivate when it ends). Each DoT tick should send an Attack + Damage pair using a separate `weaponGuid` suffixed with `-DoT` (see the DoT question under Damage). Full sequence: Attack (source weapon) → Affect Activate → Attack + Damage (DoT), Attack + Damage (DoT), ... → Affect Deactivate.

**Should grouped buffs be sent as one Affect or multiple?**

Either works. One grouped Affect is simpler. Multiple individual Affects give granular analytics per component. For most cases, one combined Affect is the practical choice.

**Should effect refreshes, extensions, or stacks trigger new events?**

Yes — send a new Affect event for refreshes, extensions, and stacks so we know the effect is still active and how it changes over time.

**How should stacking effects be represented?**

Send a new Activate for each stack. For most practical purposes, a single Activate when the effect first applies and a single Deactivate when it fully wears off is also sufficient.

**Should ability cooldowns be reported as Affects?**

No — don't track the cooldown itself. Track the ability's actual active window: send `Activate` when the ability triggers, and `Deactivate` when its effect ends. The cooldown is a side effect of when you call `Activate` and `Deactivate`. If the ability is being abused, we'll see it either from an `Activate` arriving before the previous `Deactivate`, or from the Affect triggering faster than the learned normals for that ability.

**Should passive effects (always active) be sent?**

If the passive affects movement speed, damage modifiers, or other gameplay-relevant characteristics, send an Affect Activate at spawn time. Cosmetic-only passives can be skipped.

---

## Damage

**Should all damage trigger a Damage event, including instant kills?**

Yes. Every instance of health loss generates a Damage event. An instant kill = Damage event (full damage amount) + Death event. Both are needed.

**Can Damage events exist without a preceding Attack (e.g., environment)?**

Yes. Attack and Damage are independent. Environment damage uses `"Environment"` as the `playerGuid`.

**How should Damage-over-Time (DoT) damage be represented?**

Treat DoT as its own Attack + Damage flow using a separate `weaponGuid`, suffixed with `-DoT` so the relationship to the source weapon stays explicit.

Example:
- User-triggered weapon attacks → `AK-47-Poison`
- Poison damage ticks → `AK-47-Poison-DoT`

Each DoT tick should send an Attack event followed by a Damage event. This keeps the source weapon's real usage, accuracy, and behavior stats clean, while allowing separate analytics for DoT damage, kills, and tick behavior. DoT GUIDs naturally have near-100% hit rates since they represent guaranteed damage, not aim-based attacks.

**How should overkill damage be represented?**

Send the actual damage dealt, even if it exceeds remaining health. Don't clamp to remaining health — this gives us real weapon damage data for analytics.

**Should blocked or mitigated damage trigger a Damage event?**

Send Damage only for actual health lost (post-mitigation). If all damage is fully blocked, you can skip the event or send `damageDone = 0`. Track shield/armor state via Affects.

**What should `weaponGuid` represent?**

Any damage source — weapons, abilities, traps, environmental hazards. It's a string identifier (max 36 chars). The more specific you are, the better the analytics.

**How should indirect damage (traps, projectiles, NPCs) be attributed?**

Attribute to whoever is logically responsible. Player-placed traps → owning player. Autonomous NPCs → the NPC (if tracked as a separate entity with NPC prefix) or the owning player. Environment → `"Environment"`.

**Can damage be negative or zero?**

Zero is valid (fully blocked). Negative doesn't make sense — use the Heal action for health gains.

---

## Heal

**Should Heal events be triggered for any positive health change?**

Yes. Any time a player gains health — self-heal, teammate heal, health pickup, passive regen — send a Heal event.

**How should overhealing be handled?**

Send whatever amount your game considers healed. There's no max health cap on our side — however much health you send, we add it to the player's tracked health. Just match what actually happens in your game.

**If healing is converted into damage, should it be a Damage event instead?**

Yes. Heal = health goes up, Damage = health goes down. Don't use negative values for Heal.

**Should heal-over-time be individual events or aggregated?**

Either approach works — same as damage-over-time. Use whatever fits your game's logic.

**Should shields or temporary HP be treated as Heal or Affect?**

Depends on your game's mechanics. If the shield grants bonus health, you can send it as a Heal when applied, and remove it via a Damage event with `"Environment"` when the shield expires or breaks. Pair it with an Affect (`"shield", Activate` / `"shield", Deactivate`) for additional context. Use whatever maps best to how shields work in your game.

---

## Death

**Should every death trigger a Death event, regardless of respawn mechanics?**

Yes, always. Every time a player's life ends, send Death — regardless of respawn timing. The `attackerGuid` tells us who killed them. Use `"Environment"` for PvE deaths, or the player's own guid for suicides.

**Should downed/knocked states be treated as Death?**

No. Downed/knocked means the player is still alive. Use an Affect (`"downed", Activate` → `"downed", Deactivate` when revived). Only send Death when the player is fully eliminated. The logic: if they can come back without respawning, they didn't die.

---

