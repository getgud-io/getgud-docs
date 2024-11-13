# Receiving Player Data Via Getgud.io

With Getgud.io's **Rules**, you can seamlessly receive player data in a structured JSON format via webhooks you define when certain rule conditions are met. The webhook sends the data to an endpoint of your choice, making it easy to manage and analyze players based on custom cohorts you build. One rule execution can send up to 1000 player results, and the fastest interval a rule can execute is once per hour, meaning each rule can potentially send up to 24K player results per day.

Here’s a breakdown of the JSON fields you’ll receive when a rule executes, queries player data, groups players into cohorts, and sends this data:

## JSON Response Fields

- **ruleId**: Unique identifier for the executed rule.
- **rulePK**: Primary key for the rule, unique to each rule.
- **titleId**: ID associated with the game title.
- **ruleName**: Name of the rule that triggered the webhook.
- **titleName**: Name of the game associated with the rule.

## Player Object Fields

Each player is listed as an object within the `players` array. Here’s an explanation of each field you’ll receive:

- **titleId**: The game title ID, matching the game's identifier.
- **playerId**: Unique ID for each player.
- **playerGuid**: Globally unique identifier you provided for the player.
- **playerNickname**: Player’s in-game nickname.
- **playerEmail**: Player’s email, if available.
- **playerRank**: The player’s current rank in the game.
- **playerSuspectScore**: A score between 0-100 you provided (automatically via rules or manually via updating player data), indicating the player’s likelihood of suspicious behavior.
- **playerReputation**: Reputation status of the player (e.g., "Esteemed").
- **playerStatus**: Current status (e.g., "Normal," "Banned").
- **playerTag1**-**playerTag5**: Custom tags for categorizing or labeling players.
- **lastPlayedTimeEpoch**: Last time the player played, in Unix epoch format.
- **playerJoinDateEpoch**: Player’s join date, in Unix epoch format.
- **insertTimeEpoch**: The time this player’s data was initially logged.
- **updateTimeEpoch**: Last time this player’s data was updated.
- **playerToxicityScore**: an automatic score indicating the player’s toxicity level (0-100).
- **playerTotalReportCount**: Number of reports filed against the player.
- **playerTotalMatchCount**: Total matches played.
- **totalDurationPlayedEpoch**: Total time played, in epoch milli.
- **suspendPlayerInsights**: Indicates whether insights are temporarily suspended for this player.
- **playerKillCount**: Total kills accumulated.
- **playerDeathCount**: Total deaths.
- **playerKDRatio**: Kill/Death ratio.
- **playerWinCount**: Wins accrued by the player.
- **playerLossCount**: Losses accrued.
- **playerWLRatio**: Win/Loss ratio.
- **playerHitRate**: Player’s hit rate, as a decimal (e.g., 0.34 for 34%).
- **playerCritRate**: Player’s critical hit rate.

This JSON format ensures you can integrate real-time player insights directly into your systems. By configuring rules and customizing cohorts, you can automate player engagement and management at scale, tailored precisely to your needs.

## JSON Example

```json
{
  "ruleId": 178,
  "rulePK": "d048d060-a1ff-11ef-a8b5-f19d367dbb63",
  "titleId": 133,
  "ruleName": "test-rule",
  "titleName": "CS2",
  "players": [
    {
      "titleId": 133,
      "playerId": 5992405,
      "playerGuid": "76561197996777789",
      "playerNickname": "",
      "playerEmail": null,
      "playerRank": 1,
      "playerSuspectScore": 0,
      "playerReputation": "Esteemed",
      "playerStatus": "Normal",
      "playerTag1": "",
      "playerTag2": "",
      "playerTag3": "",
      "playerTag4": "",
      "playerTag5": "",
      "lastPlayedTimeEpoch": 1731527066942,
      "playerJoinDateEpoch": null,
      "insertTimeEpoch": 1731451391038,
      "updateTimeEpoch": 1731525565519,
      "playerToxicityScore": 0,
      "playerTotalReportCount": 0,
      "playerTotalMatchCount": 42,
      "totalDurationPlayedEpoch": 3152904,
      "suspendPlayerInsights": 0,
      "playerKillCount": 24,
      "playerDeathCount": 34,
      "playerKDRatio": 0.7058823529411765,
      "playerWinCount": 18,
      "playerLossCount": 24,
      "playerWLRatio": 0.75,
      "playerHitRate": 0.13,
      "playerCritRate": 0
    },
    {
      "titleId": 133,
      "playerId": 5992407,
      "playerGuid": "76561198081879425",
      "playerNickname": "",
      "playerEmail": null,
      "playerRank": 1,
      "playerSuspectScore": 0,
      "playerReputation": "Esteemed",
      "playerStatus": "Normal",
      "playerTag1": "",
      "playerTag2": "",
      "playerTag3": "",
      "playerTag4": "",
      "playerTag5": "",
      "lastPlayedTimeEpoch": 1731527066945,
      "playerJoinDateEpoch": null,
      "insertTimeEpoch": 1731451391042,
      "updateTimeEpoch": 1731525565522,
      "playerToxicityScore": 5.5,
      "playerTotalReportCount": 0,
      "playerTotalMatchCount": 42,
      "totalDurationPlayedEpoch": 3152904,
      "suspendPlayerInsights": 0,
      "playerKillCount": 30,
      "playerDeathCount": 36,
      "playerKDRatio": 0.8333333333333334,
      "playerWinCount": 18,
      "playerLossCount": 24,
      "playerWLRatio": 0.75,
      "playerHitRate": 0.34,
      "playerCritRate": 0
    }
    // ... <up to 1000 player results>
  ]
}
