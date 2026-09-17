# Getgud Guard Laws

## Behavioral Detection & Player Safety Reference

---

## 1. Overview

Getgud Guard Laws are automated behavioral detections that continuously analyze gameplay and player activity to identify cheating, gameplay abuse, toxicity, and other behaviors that can negatively affect competitive integrity or player experience.

Unlike traditional client-side anti-cheat systems, Guard Laws operate primarily on **server-side gameplay telemetry**. They evaluate what actually happened during a match: player movement, attacks, damage, deaths, interactions, communication, and other recorded gameplay behavior.

A Guard Law does not simply answer *"Did this player cheat?"* Instead, it identifies behavior that matches a defined suspicious or abusive pattern and provides the studio with the evidence and context needed to investigate or take action.

Depending on the title and configuration, Guard Laws can be used to:

- Flag suspicious players or matches
- Prioritize cases for investigation
- Build player-behavior profiles over time
- Trigger alerts or operational workflows
- Support warnings, restrictions, suspensions, or other enforcement
- Automatically take action when the studio chooses to enable automated enforcement

Detection sensitivity and tolerance can be configured to reflect the mechanics and acceptable behavior of each individual game.

---

## 2. Guard Law Summary

| Guard Law | Category | Detects |
| --- | --- | --- |
| **Aimbot** | Anti-Cheat | Abnormal aiming and accuracy behavior consistent with automated or assisted aiming |
| **Wallhack** | Anti-Cheat | Awareness and targeting behavior consistent with obtaining information a player should not legitimately possess |
| **Speedhack** | Anti-Cheat | Movement behavior that exceeds legitimate gameplay constraints |
| **Rapid Fire** | Anti-Cheat | Firing behavior inconsistent with legitimate weapon fire-rate limitations |
| **AFK** | Gameplay Abuse | Players who are inactive or effectively not participating in the match |
| **Feeding** | Gameplay Abuse | Repeated deaths or engagements consistent with intentional feeding |
| **Intentional Team Kill** | Gameplay Abuse | Friendly damage or kills consistent with deliberate attacks against teammates |
| **Chat Moderation** | Player Safety | Toxic, abusive, threatening, hateful, or otherwise inappropriate player communication |
| **Display Name Abuse** | Identity Safety | Abusive or inappropriate player names and display identities **(upcoming)** |

---

## 3. Aimbot Guard Law

### Purpose

The Aimbot Guard Law identifies gameplay behavior consistent with the use of automated or artificially assisted aiming.

The objective is not simply to identify players with high accuracy. Highly skilled players can legitimately achieve exceptional performance. Instead, Getgud analyzes aiming and combat behavior in context and looks for patterns that are statistically or behaviorally inconsistent with normal human gameplay.

### What It Detects

The Guard Law looks for abnormal aiming and accuracy patterns associated with aim assistance or automated target acquisition.

Depending on the game and available telemetry, relevant behavior may include unusually consistent targeting performance, suspicious target acquisition, abnormal hit patterns, or other aiming behavior that differs significantly from legitimate gameplay.

### Why It Matters

Aimbots directly undermine competitive integrity and are among the most disruptive forms of cheating in competitive shooters and other aim-dependent games.

The Guard Law helps studios identify suspicious players while preserving the ability to distinguish genuine high-skill gameplay from behavior that warrants investigation.

### Investigation Context

When a detection occurs, the surrounding match can be investigated using Getgud's gameplay telemetry and replay capabilities, allowing reviewers to understand the behavior that produced the signal rather than relying on an isolated detection.

---

## 4. Wallhack Guard Law

### Purpose

The Wallhack Guard Law identifies player behavior consistent with having access to information about opponents that should not legitimately be available to the player.

This includes behavior commonly associated with wallhacks, ESP systems, or other forms of unauthorized player-location awareness.

### What It Detects

The Guard Law analyzes patterns of awareness and targeting that suggest a player may be repeatedly reacting to, tracking, positioning around, or engaging opponents without a legitimate gameplay explanation for that knowledge.

The goal is to identify **wallhack-style behavior**, rather than attempt to identify a particular cheat program or client modification.

### Why It Matters

Information-based cheats can be difficult to detect using traditional methods because the cheater may deliberately avoid obvious behavior.

Behavioral detection allows Getgud to evaluate the player's actions over time and identify suspicious patterns regardless of the specific software or technique used to obtain the unauthorized information.

### Investigation Context

Reviewers can examine the relevant player behavior inside the surrounding match context to determine whether the apparent knowledge could reasonably have come from normal gameplay information.

---

## 5. Speedhack Guard Law

### Purpose

The Speedhack Guard Law detects abnormal movement behavior that exceeds the legitimate movement constraints of the game.

### What It Detects

The Guard Law analyzes player movement and identifies situations where a player moves faster, farther, or in a manner that is inconsistent with the mechanics and conditions that should legitimately apply to that player.

The analysis can account for legitimate gameplay mechanics such as abilities, buffs, effects, character states, vehicles, or other systems that modify movement when those mechanics are represented in the game's telemetry.

### Why It Matters

Movement manipulation can provide significant competitive advantages while also disrupting the underlying rules of the game.

Server-side behavioral analysis provides a platform-independent way to identify impossible or highly anomalous movement without depending on the specific implementation of the cheat.

---

## 6. Rapid Fire Guard Law

### Purpose

The Rapid Fire Guard Law identifies firing behavior that violates or appears inconsistent with the legitimate firing characteristics of a weapon, ability, or other attack mechanism.

### What It Detects

The Guard Law evaluates attack timing and firing patterns to identify behavior consistent with artificially increased fire rates or automated firing.

Examples can include attacks occurring faster than the permitted fire rate or sustained firing patterns that cannot reasonably be produced under the weapon's legitimate gameplay configuration.

The detector is evaluated in the context of the weapon and gameplay state rather than applying a single universal firing threshold.

### Why It Matters

Rapid-fire manipulation can significantly alter damage output and competitive balance.

Because Getgud evaluates the resulting gameplay behavior, the Guard Law can identify suspicious firing patterns regardless of whether they originate from a modified client, macro, exploit, or another mechanism.

---

## 7. AFK Guard Law

### Purpose

The AFK Guard Law identifies players who remain in a match but are no longer meaningfully participating in gameplay.

### What It Detects

The Guard Law evaluates player activity over time and identifies extended inactivity or behavior consistent with a player abandoning active participation.

AFK detection is not limited to whether the game client remains connected. A player may technically remain connected to the server while effectively contributing nothing to the match.

### Why It Matters

AFK players can significantly degrade match quality, particularly in team-based competitive games.

Detecting AFK behavior allows studios to identify repeat offenders, understand the impact on other players, and implement appropriate matchmaking, progression, moderation, or enforcement policies.

---

## 8. Feeding Guard Law

### Purpose

The Feeding Guard Law identifies behavior consistent with a player intentionally allowing themselves to be repeatedly killed or otherwise deliberately providing an advantage to the opposing team.

### What It Detects

The Guard Law evaluates death and engagement behavior over the course of a match and looks for patterns that may indicate intentional feeding.

Importantly, **poor performance alone is not feeding**.

A legitimate player can have a bad match, make mistakes, or repeatedly lose engagements. The purpose of the Guard Law is to distinguish ordinary gameplay outcomes from repeated behavioral patterns that warrant investigation for intentional abuse.

### Why It Matters

Intentional feeding can:

- Ruin competitive matches
- Manipulate ranking systems
- Assist opponents
- Facilitate boosting or coordinated abuse
- Create a highly negative experience for teammates

Behavioral analysis allows feeding to be evaluated using the context of the full match rather than relying on a simple number-of-deaths threshold.

---

## 9. Intentional Team Kill Guard Law

### Purpose

The Intentional Team Kill Guard Law identifies players who deliberately damage or kill teammates.

### What It Detects

The Guard Law evaluates friendly-fire behavior and looks for patterns consistent with intentional attacks against teammates.

A friendly kill by itself does not necessarily indicate abuse. Games may contain accidental friendly fire, area-of-effect damage, chaotic combat situations, or other mechanics that can legitimately result in teammate damage.

The Guard Law therefore evaluates the behavior in context and over time rather than automatically treating every friendly-fire incident as intentional griefing.

### Why It Matters

Intentional team killing is one of the clearest forms of gameplay sabotage. It can destroy match quality and is frequently associated with griefing, retaliation, harassment, or coordinated abusive behavior.

The Guard Law gives studios a scalable method of identifying recurring or severe friendly-fire behavior while reducing the likelihood of treating legitimate accidents as intentional abuse.

---

## 10. Chat Moderation Guard Law

### Purpose

The Chat Moderation Guard Law identifies toxic, abusive, or otherwise inappropriate communication between players.

Getgud uses AI-powered language analysis rather than relying exclusively on traditional keyword lists. This enables the system to interpret language in context and operate across multiple languages.

### What It Detects

Depending on the moderation policy configured for the title, the system can identify communication associated with toxicity, harassment, threats, hate or identity-based abuse, sexual or inappropriate content, and other forms of harmful communication.

Messages receive a toxicity assessment that can be used to determine how the game responds.

For real-time moderation workflows, Getgud can provide an immediate moderation decision before the message is delivered, including the ability to:

- **Allow** the original message
- **Sanitize** inappropriate portions of the message
- **Block** the message

The studio retains control over the final moderation and enforcement policy.

### Why It Matters

Chat toxicity is rarely best understood from a single keyword or isolated message.

AI-based contextual analysis enables more accurate moderation while helping reduce false positives caused by simple keyword filtering. Player behavior can also be evaluated over time, allowing studios to differentiate isolated incidents from recurring toxic behavior.

---

## 11. Display Name Abuse Guard Law

> **Status:** Upcoming / Planned

### Purpose

The Display Name Abuse Guard Law extends Getgud's player-safety capabilities beyond gameplay and chat to the identities players present inside the game.

### What It Detects

The Guard Law analyzes player display names for abusive, offensive, hateful, sexually inappropriate, threatening, or otherwise prohibited content.

Because abusive names frequently use deliberate misspellings, substitutions, spacing, numbers, symbols, or other forms of obfuscation, the system is designed to evaluate the meaning of the name rather than relying only on exact banned-word matching.

### Why It Matters

A player's display name can be visible across matchmaking, leaderboards, social systems, chat, replays, and other areas of the game.

Identifying abusive names automatically helps studios protect these surfaces without requiring every variation of inappropriate terminology to be manually maintained in a blacklist.

### Status

**Upcoming / planned capability.**

The capability should be treated separately from currently deployed Guard Laws until enabled for the applicable title.

---

## 12. Behavioral Detection vs. Automatic Enforcement

A Guard Law detection and an enforcement action are intentionally separate concepts.

A detection indicates that Getgud observed behavior consistent with the conditions the Guard Law is designed to identify.

What happens next is controlled by the game studio.

Depending on the title, confidence requirements, and enforcement policy, a detection can be used to:

- Add the player to an investigation queue
- Flag the relevant match
- Review the behavior in replay
- Accumulate behavioral history
- Issue a warning
- Restrict functionality
- Adjust matchmaking
- Suspend or ban a player
- Trigger another title-specific workflow

Studios can therefore begin by using Guard Laws as investigation and observability tools, validate their behavior against real gameplay, and progressively introduce automated enforcement where appropriate.

---

## 13. Context Matters

Getgud's philosophy is that behavioral detections should be interpreted in the context of the actual game.

A single unusual event does not automatically make a player a cheater or griefer.

Character abilities, weapons, buffs, map conditions, game modes, skill level, server conditions, and title-specific mechanics can all affect player behavior.

For this reason, Guard Laws are designed to work together with Getgud's broader gameplay observability capabilities, including:

- Complete match telemetry
- Player timelines
- 2D and 3D replay
- Action-level investigation
- Match and player filtering
- Behavioral history
- Analytics and supporting evidence

This allows teams to move from:

> **Detection → Evidence → Investigation → Decision**

rather than relying on an unexplained binary verdict.

---

## 14. Title-Specific Configuration

Guard Laws are not intended to impose identical behavioral rules on every game.

Every title has different:

- Movement mechanics
- Weapon behavior
- Character abilities
- Team structures
- Match lengths
- Game modes
- Friendly-fire rules
- Skill distributions
- Player communities
- Moderation policies

Getgud works with the studio to validate Guard Laws against the title's actual gameplay and configure appropriate tolerances before they are used for production enforcement.

Additional title-specific Guard Laws and behavioral detections can also be introduced as new gameplay or operational requirements emerge.

---

## Summary

Guard Laws convert gameplay telemetry into actionable player-integrity signals.

They provide studios with a unified framework for detecting:

### Cheating

**Aimbot · Wallhack · Speedhack · Rapid Fire**

### Gameplay Abuse

**AFK · Feeding · Intentional Team Killing**

### Communication & Identity Abuse

**Chat Moderation · Display Name Abuse**

Together with Getgud's replay and gameplay observability capabilities, Guard Laws allow teams not only to identify suspicious behavior, but to understand **what happened, why it was detected, and what evidence exists before action is taken.**
