# Cosmos DB Containers, Partition Keys, and Composite Indexes

## Container Overview

|Container	|Partition Key	|Composite Indexes	|Reasoning                 |
|:----------|:--------------|:------------------|:-------------------------|
|Users	    |/id (userId)	|Default	        |Users evenly distributed.|
|Teams	    |/id (teamId)	|Default	        |Unique per team; balanced partitions.|
|TeamMembers|/userId	    |[["teamId", 1]]	|Quickly retrieve teams per user; supports ordering/filtering.|
|Groups 	|/teamId	    |Default	        |Efficiently retrieve all groups under a team.|
|Players	|/id (playerId)	|Default	        |High cardinality; even data distribution.|
|Stats	    |/playerId	    |[["season", -1]]	|Efficient retrieval of recent season stats per player.|
|Projections|/playerId	    |[["model", 1], ["year", -1]]	|Quick queries for projections per player by model and year.|
|NewsArticles|/id (articleId)	|[["vibe", 1], ["publishedOn", -1]]	|Rapid queries for positive/negative articles by recency.|
|Notes	    |/targetId	|[["targetType", 1], ["createdOn", -1]]	|Quickly filter notes per target and creation date.|

## Sample Document Schemas

### User
```json
{
  "id": "user100123",
  "name": "Joe User",
  "email": "joe@example.com",
  "createdOn": "2025-01-01T12:00:00Z"
}
```
### Teams
```json
{
  "id": "team500001",
  "name": "Champions",
  "ownerId": "user100123",
  "sharedWith": ["user500001", "user750001"],
  "isPrivate": false,
  "createdOn": "2025-02-10T10:00:00Z"
}
```
### TeamMembers
```json
{
  "id": "membership900001",
  "userId": "user100123",
  "teamId": "team500001",
  "role": "owner",
  "joinedOn": "2025-02-10T10:01:00Z"
}
```
### Groups
```json
{
  "id": "group700001",
  "teamId": "team500001",
  "name": "Outfielders Watchlist",
  "playerIds": ["player10001", "player10002", "..."],
  "createdOn": "2025-03-01T09:00:00Z"
}
```
### Players
```json
{
  "id": "player10001",
  "name": "John Doe",
  "position": "OF",
  "team": "Yankees",
  "active": true
}
```
### Stats
```json
{
  "id": "stats-player10001-2024",
  "playerId": "player10001",
  "season": 2024,
  "homeRuns": 40,
  "atBats": 500,
  "lastUpdated": "2025-05-09T23:59:59Z"
}
```
### Projections
```json
{
  "id": "proj-player10001-steamer-2025",
  "playerId": "player10001",
  "model": "Steamer",
  "year": 2025,
  "homeRuns": 38,
  "atBats": 480,
  "lastUpdated": "2025-05-09T23:59:59Z"
}
```
### NewsArticles
```json
{
  "id": "article90001",
  "title": "Doe Hits Walk-Off Homer",
  "content": "John Doe smashed a walk-off home run...",
  "playerIds": ["player10001"],
  "vibe": "positive",
  "source": "ESPN",
  "publishedOn": "2025-05-09T20:00:00Z"
}
```
### Notes
```json
{
  "id": "note800001",
  "authorId": "user100123",
  "content": "Expect breakout performance this year",
  "targetType": "player",
  "targetId": "player10001",
  "createdOn": "2025-05-10T12:00:00Z"
}
```
## Sample Cosmos DB SQL Queries

### Fetch all teams a user belongs to (private or shared)
```sql
SELECT tm.teamId, tm.role
FROM TeamMembers tm
WHERE tm.userId = @userId
ORDER BY tm.teamId ASC
```
### Get all players in a group
```sql
SELECT g.playerIds
FROM Groups g
WHERE g.teamId = @teamId AND g.id = @groupId
```
### Retrieve stats by season for a player
```sql
SELECT *
FROM Stats s
WHERE s.playerId = @playerId AND s.season = @season
```
### Fetch projections for a player by model and year
```sql
SELECT *
FROM Projections p
WHERE p.playerId = @playerId 
  AND p.model = @model 
  AND p.year = @year
```
### List notes for a player or group (recent first)
```sql
SELECT *
FROM Notes n
WHERE n.targetId = @targetId
  AND n.targetType = @targetType
ORDER BY n.createdOn DESC
```
### $Find all positive articles mentioning a specific player (recent first)
``` sql
SELECT *
FROM NewsArticles n
WHERE ARRAY_CONTAINS(n.playerIds, @playerId)
  AND n.vibe = 'positive'
ORDER BY n.publishedOn DESC
```