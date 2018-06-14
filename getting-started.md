# Introduction
The purpose of this document is to provide SMITE API Developers and PALADINS API Developers with the necessary information to access and utilize the API methods. The JSON data returned from the API methods is designed to be "self-documenting" and therefore detailed descriptions of each data field are not included in this API Developer Guide.

# Registration
To register to become developer, go [here](https://fs12.formsite.com/HiRez/form48/secure_index.html) to register. If your application is accepted you will receive custom credentials to access the API.

# Credentials
To access the APIs you'll need your own set of credentials which consist of a developer id (devId) and an authentication key (authKey).  The process of obtaining credentials should be a one-time activity.

Here are the credentials for a sample account:

* DevId: (eg, 1004)
* AuthKey: (eg, 23DF3C7E9BD14D84BF892AD206B6755C)

Use your personal credentials to access the api via a *Representational State Transfer* (**REST**) web service hosted at api.smitegame.com. Note that the same DevId/AuthKey combination should work for both SmiteAPI and PaladinsAPI, across all supported platforms.

# Sessions
To begin using the API, you will first need to establish a valid Session. To do so you will start a session (via the `/createsession` method) and receive a `SessionId`. Sessions are used for authentication, security, monitoring, and throttling. Once you obtain a `SessionId`, you will pass it to other methods for authentication. Each session only lasts for **15 minutes** and must be recreated afterward.

# API Access Limits
To throttle API Developer access various limits have been setup to prevent over use of the API (either intentional, more likely unintentional “over use”).

Here are the default initial limitations for API Developers:

```yaml
concurrent_sessions: 50
sessions_per_day: 500
session_time_limit: 15 minutes
request_day_limit: 7500
```

# Platform Endpoint Base URLs
As of version  2.11, the SmiteAPI and Paladins API supports 3 platforms: `PC`, `Xbox` & `PS4`. Each platform is supported by a separate endpoint. Applications access each endpoint via a unique base URL.

The interface for each endpoint is identical. Therefore, a single application code-base should be able to support all supported platforms.

However, since each endpoint is serviced by a unique Web Service, applications must create and maintain separate Sessions for each endpoint. In other words, you cannot use a Session ID created by the SmiteAPI for PC endpoint (via the `/createsession` method) to make method calls to the SmiteAPI for Xbox endpoint.

Note that, depending on the Hi-Rez development pipeline and platform certification requirements, some SmiteAPI features will not be immediately available in all platforms. For instance, if Xbox does not currently support Clans, then clan/team-related methods should return empty datasets or an error response.

# SmiteAPI for Xbox Special Considerations
As stated earlier, the **SmiteAPI for Xbox** endpoint supports the same API interface as does **SmiteAPI for PC**. However, supporting applications must be aware of the following differences that might lead to unexpected responses:

1. Players of **SMITE for Xbox** are identified by **Xbox Live** `GamerTags` instead of Smite player names. Therefore, when searching for player information, you should specify `GamerTag` as a parameter where you would normally specify playername. By the same token, API method results will return **Xbox Live** `GamerTag` as `playername`, as is expected in **SmiteAPI for PC**.
2. **Xbox Live** `Gamertags` are owned and administered by Microsoft, outside the purview of Hirez Studios. We cache current player GamerTag each time a player logs into **SMITE for Xbox** and invalidate any other players that might have the same cached `GamerTag`. In this way, we hope to keep `GamerTag` searches as current as possible. However, it is possible that stored `GamerTag` names become *stale* when players change `GamerTag` through Microsoft, especially if said player does not log into **SMITE for Xbox** often. By the same token, if an abandoned GamerTag is transferred to another Xbox Live player, it is possible that a **SmiteAPI** query could return data for the wrong player. Hopefully, this is not a common occurrence, but we wanted to notify you of the possibility. Note that **SMITE** `playerId` is guaranteed unique. Therefore, if your application already stores **SMITE** `playerId` of customers and specifies that value instead of `GamerTag`, you eliminate the possibility of name collisions.
3. Caching of `GamerTags` has only been implemented fairly recently.  Therefore, searches of players by `GamerTag` who have not logged into **SMITE for Xbox** since caching was implemented will obviously fail.

# SMITE Player Privacy Option
**SMITE** version **2.14** has introduced the option of player opt-in privacy.  Players choosing to enable privacy for their account will affect data returned by **SmiteAPI** for said players:

1. Player queries on individual players marked `Private` will fail. In particular, query methods should return empty dataset, just as if the player did not exist. For example, a `/getfriends` call on the player `JohnDoe` would return an empty dataset if JohnDoe was marked Private.
2. Methods that return data on multiple players may still return some data for Private players, while other data is obfuscated. In particular, for private players:
    * Player `Name`: returned as `""` (empty string)
    * PlayerId: returned as 0 (int value zero) or `"0"` (string), depending on method
    * ClanId: returned as 0 (int value zero)

As an example for item `#2` above, the `/getmatchdetails` method for a specified MatchId would return an object for each player in the Match.  However, the returned object for each Private player would include a `playerName` of `""` and a `playerId` of `0`.