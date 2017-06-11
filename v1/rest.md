# Crude Cards - Protocol v1 _(Draft)_
See also: [Gateway](gateway.md) and [Structures](structures.md)

## REST API

1. Responses must be JSON.
2. 🔒 indicates that an endpoint requires a valid `authorization` header. If this header is invalid or not present, the server should respond with the code 401.
3. (🔒) indicates that authorization may not always be required (see endpoints for specific information.)
4. `?` indicates that a parameter is optional.
5. All rate-limits provided in this specification are _suggested_ - you can change them if you wish.
6. Provided rate-limits are in the format `limit/time - mode`, where `time` is the time in seconds, e.g. 5/60 means 5 requests per minute. If `mode` is `token`, then the rate-limits should be applied by the `authorization` header and not the IP address.
7. All requests made to the server will use HTTPS.
8. Mentioned structures, e.g. _"User object"_, can be found in the structures documentation (link provided above)

-----
## Default Responses
### 401 - Invalid Authorization
For endpoints that require authorization and invalid authorization is given:
```js
{
  "message": "Authorization required."
}
```

### 404 - Resource Not Found
If applicable, use an endpoint's specific 404 response, otherwise respond with the following JSON and error code 404:
```js
{
  "message": "Resource not found."
}
```

### 429 - Rate-limiting
To rate-limit an endpoint, make sure the following headers are present:

#### Headers
- `X-RateLimit-Limit` - number of requests allowed in the current window.
- `X-RateLimit-Remaining` - number of requests remaining in the current window.
- `X-RateLimit-Reset` - The remaining window before the rate limit resets (in UTC epoch seconds.)
- `Retry-After` - Time remaining (milliseconds) until the current window resets (used to minify time offsets.)

#### Response (429)
```js
{
  "message": "You are being rate-limited.",
  "meta": { // the meta field is optional, but if provided it MUST contain these properties.
    "limit": 0,
    "remaining": 0,
    "reset": 0,
    "retryAfter": 10
  }
}
```

-----

## Endpoints Overview
- Meta and Authentication
  - `GET /api/meta`
  - `POST /api/authenticate`
  - `POST /api/authenticate/discord`
  - `POST /api/authenticate/google`
- Integrations
  - `🔒 POST /api/integrations/discord`
  - `🔒 DELETE /api/integrations/discord`
  - `🔒 POST /api/integrations/google`
  - `🔒 DELETE /api/integrations/google`
- Users
  - `🔒 GET /api/users/:id`
  - `🔒 PATCH /api/users/:id`
- Games
  - `🔒 GET /api/games`
  - `🔒 POST /api/games`
  - `(🔒) GET /api/games/:id`
  - `🔒 PATCH /api/games/:id`
  - `🔒 DELETE /api/games/:id`
  - `🔒 POST /api/games/:id/join`
  - `🔒 POST /api/games/:id/rounds`
  - `(🔒) GET /api/games/:id/rounds`
  - `(🔒) GET /api/games/:id/rounds/:round`
  - `🔒 POST /api/games/:id/rounds/:round/cards`
  - `🔒 POST /api/games/:id/rounds/:round/winner`
  - `(🔒) GET /api/games/:id/cards`
- Messaging
  - `🔒 GET /api/messages/:channel`
  - `🔒 POST /api/messages/:channel`
- Decks
  - `GET /api/decks` (search)
  - `🔒 POST /api/decks`
  - `GET /api/decks/:id`
  - `🔒 DELETE /api/decks/:id`

-----
## Meta and Authentication
### `GET /api/meta`
#### Rate-limiting
- 5/60
#### Response (200)
```js
{
  "name": "Test Server",
  "description": "Description of Server",
  "apiVersion": 1,
  "players": 0,
  "maxPlayers": 16
}
```

### `POST /api/authenticate`
#### Rate-limiting
- 5/60
#### Parameters
- `?username` - string, alphanumeric, up to 20 characters. Server should generate a random username if one is not present.
#### Response (200)
```js
{
  "user": {}, // ExtendedUser object
  "token": "abc"
}
```

### `POST /api/authenticate/discord`
#### Rate-limiting
- 5/60
#### Parameters
- `code` - string, OAuth code from Discord.
#### Response (200)
```js
{
  "user": {}, // ExtendedUser object
  "token": "abc"
}
```
#### Response (401, bad code or cannot fetch user details using token)
```js
{
  "message": "Bad authorization code provided"
}
```

### `POST /api/authenticate/google`
#### Rate-limiting
- 5/60
#### Parameters
- `code` - string, OAuth code from Google.
#### Response (200)
```js
{
  "user": {}, // ExtendedUser object
  "token": "abc"
}
```
#### Response (401, bad code or cannot fetch user details using token)
```js
{
  "message": "Bad authorization code provided"
}
```

-----
## Integrations
### `🔒 POST /api/integrations/discord`
#### Parameters
- `code` - string, OAuth code from Discord.
#### Response (200)
```js
{
  "user": {}, // ExtendedUser object
}
```
#### Response (401, bad code or cannot fetch user details using token)
```js
{
  "message": "Bad authorization code provided"
}
```

### `🔒 DELETE /api/integrations/discord`
#### Response (201)
```js
// success
```
#### Response (404, no discord integration)
```js
{
  "message": "No Discord integration to delete"
}
```

### `🔒 POST /api/integrations/google`
#### Parameters
- `code` - string, OAuth code from Google.
#### Response (200)
```js
{
  "user": {}, // ExtendedUser object
}
```
#### Response (401, bad code or cannot fetch user details using token)
```js
{
  "message": "Bad authorization code provided"
}
```

### `🔒 DELETE /api/integrations/google`
#### Response (201)
```js
// success
```
#### Response (404, no Google+ integration)
```js
{
  "message": "No Google+ integration to delete"
}
```

-----
## Users
### `🔒 GET /api/users/:id`
#### Rate-limiting
- 10/10
#### Response (200)
```js
{
  "profile": {} // Profile object. `user` property is an ExtendedUser if client's id matches requested ID.
}
```

#### Response (404)
```js
{
  "message": "User not found."
}
```

## Users
### `🔒 PATCH /api/users/:id`
#### Parameters
- `?username` - string, alphanumeric, up to 20 characters.
- `?bio` - string, up to 120 characters.
#### Rate-limiting
- 10/10
#### Response (200)
```js
{
  "profile": {} // Profile object. `user` property is an ExtendedUser if client's id matches requested ID.
}
```

#### Response (400, bad username or bio)
```js
{
  "message": "Bad username or bio."
}
```

#### Response (403)
```js
{
  "message": "You can only edit your own profile."
}
```

-----
## Games
### `🔒 GET /api/games`
#### Rate-limiting
- 5/10 token
#### Parameters
- `?joinable` - boolean, defaults to false. If true, only games that can be joined (i.e. not full) are displayed

#### Response (200)
```js
{
  "games": [] // array of Game objects
}
```

### `🔒 POST /api/games`
#### Rate-limiting
- 1/30 token
#### Parameters
- `name` - string, up to 32 characters.
- `?password` - if present, game is private.
- `max_players` - integer, up to 32.
- `max_spectators` - integer, up to 32.
- `rules` - object containing following properties. At least `maxScore` or `maxRounds` must be enabled.
  - `max_score` - integer, up to 100. -1 to disable.
  - `max_rounds` - integer up to 100. -1 to disable.
  - `max_round_Time` - integer up to 600. -1 to disable.
  - `czar` - integer, 0 = winner, 1 = random. Other values are invalid.
- `decks` - Array of Deck IDs. At least 1 deck is required.

#### Response (200)
```js
{
  "game": {} // Game object
}
```

#### Response (400, incorrect details)
```js
{
  "message": "Incorrect details were provided."
}
```

#### Response (400, owner already in a game)
```js
{
  "message": "You are already in a game."
}
```

#### Response (503, too many games already exist)
```js
{
  "message": "Too many games are being played."
}
```

### `(🔒) GET /api/games/:id`
The endpoint should _only_ require authentication if the game is private. In this case, the endpoint should only continue the request if the authenticated user is in the game.
#### Rate-limiting
- 20/60

#### Response (200)
```js
{
  "game": {} // game object
}
```

#### Response (401)
```js
{
  "message": "You are not authorized to view this game."
}
```

### `🔒 PATCH /api/games/:id`
#### Rate-limiting
- 1/10 token

Only the owner of a game is allowed to call this endpoint.

#### Parameters
- Same parameters provided to `🔒 POST /api/games`, but additionally:
- `?owner` - integer, user ID of new owner (must be a player, otherwise 400)

#### Response (200)
```js
{
  "game": {} // Game object
}
```

#### Response (400, incorrect details)
```js
{
  "message": "Incorrect details were provided."
}
```

#### Response (401, user is not owner)
```js
{
  "message": "You are not the owner of this game."
}
```

#### Response (404, game does not exist)
```js
{
  "message": "Game does not exist."
}
```

### `🔒 DELETE /api/games/:id`
#### Rate-limiting
- 1/10 token

Deletes/leaves the current game. The owner can end the game by calling this endpoint with no parameters, or can leave and pass on ownership using the `owner` parameter (see below.)

#### Parameters
- `?owner` - integer, user ID of new owner (must be a player, otherwise 400.) Can only be specified by the current owner. If specified, an existing player is made owner and the user calling the endpoint is removed from the game. Otherwise, if the owner leaves this parameter out, the game is ended.

#### Response (204)
```js
// successful request, no content
```

#### Response (400, user is not a player)
```js
{
  "message": "The specified user is not a player."
}
```

#### Response (401, user is not owner (only if `?owner` supplied))
```js
{
  "message": "You are not the owner of this game."
}
```

#### Response (404, game does not exist)
```js
{
  "message": "Game does not exist."
}
```

#### Response (409, new owner is owner making request)
```js
{
  "message": "You cannot make yourself owner and leave."
}
```

### `🔒 POST /api/games/:id/join`
#### Rate-limiting
- 3/10 token

Joins the specified game.

#### Parameters
- `?password` - string, required if the game is private.
- `?spectator` - boolean, if true then join as a spectator.

#### Response (200)
```js
{
  "game": {} // Game object
}
```

#### Response (400, game is full)
```js
{
  "message": "Game is full."
}
```

#### Response (401, incorrect password)
```js
{
  "message": "Incorrect password provided."
}
```

#### Response (404, game does not exist)
```js
{
  "message": "Game does not exist."
}
```

### `🔒 POST /api/games/:id/rounds`
Can only be used by the game owner, starts the game.
#### Response (201)
```js
// success
```

#### Response (400, game already started)
```js
{
  "message": "The game has already started."
}
```

#### Response (401, user not owner)
```js
{
  "message": "You do not have authorization to start this game."
}
```

#### Response (404, game does not exist)
```js
{
  "message": "Game does not exist."
}
```

### `(🔒) GET /api/games/:id/rounds`
Only requires authorization if the game is private.
#### Rate-limiting
1/10
#### Parameters
- `?rounds` - can be an array of integers:
  - `[1,4,5]` would only select the second, fifth and sixth rounds.
  - `[-1, -2, -3]` would return the last 3 rounds.
  - If no value is given, all rounds are returned.
#### Response (200)
```js
{
  "rounds": [] // array of Round objects
}
```
#### Response (200, invalid round selection defaults to all rounds)
```js
{
  "rounds": [] // array of Round objects
}
```
#### Response (401, user does not have authorization to view this game)
```js
{
  "message": "You do not have authorization to view this game"
}
```
#### Response (404, game does not exist)
```js
{
  "message": "Game does not exist."
}
```

### `(🔒) GET /api/games/:id/rounds/:round`
- Only requires authorization if the game is private.
- `:round` can be:
  - an integer, the round number.
  - a string, `current` to get the current round.
#### Rate-limiting
1/10
#### Response (200)
```js
{
  "round": {} // Round object
}
```
#### Response (400, invalid round selection)
```js
{
  "message": "Invalid round selection"
}
```
#### Response (401, user does not have authorization to view this game)
```js
{
  "message": "You do not have authorization to view this game"
}
```
#### Response (404, round does not exist)
```js
{
  "message": "Round does not exist."
}
```

### `🔒 POST /api/games/:id/rounds/:round/cards`
Players call this to put forward their choice of cards.

#### Rate-limiting
- 1/10 token

#### Parameters
- `cards` - an array containing card IDs.

#### Response (400, round has ended)
```js
{
  "message": "The round is not active."
}
```

#### Response (400, cards already put forward)
```js
{
  "message": "You have put cards forward already."
}
```

#### Response (400, invalid selection of cards, e.g. cards player doesn't have)
```js
{
  "message": "Invalid card selection."
}
```

#### Response (404, game or round do not exist)
```js
{
  "message": "Game or Round do not exist."
}
```

#### Response (201)
```js
// card(s) successfully put forward
```

### `🔒 POST /api/games/:id/rounds/:round/winner`
Called by the czar to select a winner of the current round. The round should be ended immediately after the czar picks the winner.

#### Rate-limiting
- 1/10 token

#### Parameters
- `player` - integer, ID of winning player

#### Response (400, round has ended)
```js
{
  "message": "That round is not active."
}
```

#### Response (400, user hasn't put cards forward)
```js
{
  "message": "That user hasn't put any cards forward."
}
```

#### Response (403, user not czar)
```js
{
  "message": "You are not the czar."
}
```

#### Response (404, game or round do not exist)
```js
{
  "message": "Game or Round do not exist."
}
```

#### Response (201)
```js
// winner chosen
```

### `(🔒) GET /api/games/:id/cards`
- Authorization only required if game is private.
- Returns the overall set of cards that are used in this game. Only accessible once the game has started. Once the game has started, the server should compile the decks and generate unique IDs for the cards.
#### Rate-limiting
1/10
#### Response (200)
```js
{
  "cards": [] // Array of Card objects
}
```
#### Response (401, user not part of game)
```js
{
  "message": "You aren't part of this game."
}
```
#### Response (404, game does not exist)
```js
{
  "message": "Game does not exist."
}
```

-----
## Messaging
### `🔒 GET /api/messages/:channel`
Responds with the last 50 messages sent in this channel. Channel can either be -1 (global chat) or a game ID.

#### Rate-limiting
- 1/10 token, different for each channel

#### Response (200)
```js
{
  "messages": [] // array of up to 50 messages
}
```

#### Response (404, channel does not exist)
```js
{
  "message": "Channel does not exist."
}
```

#### Response (403, user does not have access to channel)
```js
{
  "message": "You do not have access to this channel."
}
```

### `🔒 POST /api/messages/:channel`
Posts a message to this channel.

#### Rate-limiting
- 10/10 token, different for each channel.

#### Parameters
- `content` - string, up to 1,000 characters - make sure to sanitise!

#### Response (200)
```js
{
  "message": {} // Message object
}
```

#### Response (400, invalid content)
```js
{
  "message": "Invalid content (too long?)"
}
```

#### Response (403, player doesn't have access to this channel)
```js
{
  "message": "You do not have access to this channel."
}
```

#### Response (404, channel does not exist)
```js
{
  "message": "Channel does not exist."
}
```

-----
## Decks
### `GET /api/decks`
TO DO.

### `🔒 POST /api/decks`
Create a new deck.

#### Rate-limiting
- 1/30 token

#### Parameters (normal usage)
- `?cardcast_id` - string, up to 8 characters. If present, no other parameters can be supplied.
- `name` - string, up to 32 characters.
- `cards` - array of objects containing the following properties (the server should generate the ID of the cards when a game starts)
  - `type` - number, 0 = black (call), 1 = white (response)
  - `text` - string, up to 180 characters.

#### Response (200)
```js
{
  "deck": {} // deck object
}
```

#### Response (400, invalid data)
```js
{
  "message": "Invalid data."
}
```

#### Response (404, card cast deck not found)
```js
{
  "message": "Deck not accessible on Card Cast."
}
```

#### Response (500, not accepting new decks for whatever reason)
```js
{
  "message": "Could not create deck at this time."
}
```

### `GET /api/decks/:id`

#### Rate-limiting
- 60/60 token

#### Response (200)
```js
{
  "deck": {} // deck object
}
```

#### Response (404, deck does not exist)
```js
{
  "message": "Deck does not exist."
}
```

### `🔒 DELETE /api/decks/:id`

#### Rate-limiting
- 1/10 token

#### Response (201)
```js
// deck deleted
```

#### Response (401, user is not creator of deck)
```js
{
  "message": "You are not the owner of this deck."
}
```

#### Response (404, deck does not exist)
```js
{
  "message": "Deck does not exist."
}
```
