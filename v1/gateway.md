# Crude Cards - Protocol v1 _(Draft)_
See also: [REST API](rest.md) and [Structures](structures.md)
## WebSocket Gateway

1. All connections will be over `wss` (secure) - `ws` is not supported.
2. Clients are only allowed to connect through `/gateway`.
3. All packets must be valid JSON.
4. Mentioned structures, e.g. _"User object"_, can be found in the structures documentation (link provided above).

-----
## Simple flow
1. Client connects to server.
2. Client has 15 seconds to send an `IDENTIFY` payload.
3. Client receives `READY` payload, detailing user details and heartbeat timings.
4. Server should specifically begin to send `GAME` packets to the user to populate their view.
5. Client begins to receive any other packets.
6. Client must periodically send a `HEARTBEAT` packet.

-----
## Packet Overview
### Meta
0. `IDENTIFY`
    - Sent by client.
    - Authenticates the client.
1. `HEARTBEAT`
    - Sent periodically by client.
    - Intervals determined by value set in `READY` packet.
2. `READY`
    -	Sent by server.
    - Contains user's details and heartbeat timings.
### Games
3. `GAME_CREATE`
    -	Sent by server.
    - Does not necessarily mean a game has _just_ been create, the client may just have connected to the gateway and is receiving information of existing games.
4. `GAME_UPDATE`
    - Sent by server.
    - Only sent if there are changes to the configuration of a game, not if there is a new round or winner.
5. `GAME_PLAYERS_UPDATE`
    - Sent by server when a player joins/leaves a game.
6. `GAME_DELETE`
    -	Sent by server once a game has been deleted or ended.
7. `GAME_ROUND_START`
    -	Sent by server when a new round in a game has started.
8. `GAME_ROUND_END`
    -	Sent by server when the current round in a game has ended.
### Users
9. `USER_CREATE`
    -	Sent by server.
    - Does not necessarily mean a user has just joined the gateway.
10. `USER_UPDATE`
    - Sent when a user updates their details.
### Messaging
11. `MESSAGE_CREATE`
    - Sent when a message is created.

-----

## Meta Packets
### `IDENTIFY`
- Sent by client.
- If the token is invalid, the server should disconnect the client from the gateway with code 1000.
#### Example
```js
{
  "t": 0,
  "d": {
    "token": "abc123",
    "resume": true // if true, indicates the client wants to resume an existing connection
  }
}
```

### `HEARTBEAT`
- Sent by client
- Gateway should set a reasonable heartbeat period (e.g. 60 seconds), if heartbeats are not received within this period, the client should be disconnected with code 1000.
#### Example
```js
{
  "t": 1
}
```

### `READY`
- Sent by server once the client has been successfully authenticated.
#### Example
```js
{
  "t": 2,
  "d": {
    "heartbeat": 60000,
    "user": {}, // User object
    "resumed": true, // whether or not the session could be resumed
    // following only present if resumed
    "resume_data": {
      "game": {}, // Game object, present if user was in a game
      "cards": [] // User's cards
    }
  }
}
```

-----
## Game Packets

### `GAME_CREATE`
- Sent by server.
- Once a client authenticates, they should be sent ~20 games.
- Also sent if a new public game has been created.
#### Example
```js
{
  "t": 3,
  "d": {
    "game": {} // Game object
  }
}
```

### `GAME_UPDATE`
- Sent by server.
- Only sent if any properties of the normal Game object change, except `players` or `spectators` - they use `GAME_PLAYERS_UPDATE`.
#### Example
```js
{
  "t": 4,
  "d": {
    "game": {} // Game object
  }
}
```

### `GAME_PLAYERS_JOIN`
- Sent by server.
- Only sent if new players/spectators join a game.
#### Example
```js
{
  "t": 5,
  "d": {
    "game": { // Partial game object
      "id": 123,
      "players": [], // New players
      "spectators": [] // New spectators
    }
  }
}
```

### `GAME_PLAYERS_LEAVE`
- Sent by server.
- Only sent if players/spectators of a game leave.
#### Example
```js
{
  "t": 5,
  "d": {
    "game": { // Partial game object
      "id": 123,
      "players": [], // Players that have left
      "spectators": [] // Spectators that have left
    }
  }
}
```

### `GAME_DELETE`
- Sent by server.
- Sent once a game has ended or been deleted.
#### Example
```js
{
  "t": 6,
  "d": {
    "game": { // Partial game object
      "id": 123
    }
  }
}
```

### `GAME_ROUND_START`
- Sent by server when a new round has started.

#### Example
```js
{
  "t": 7,
  "d": {
    "round": {}, // Round object
    "cards": [] // The player's current cards (Array of Card objects)
  }
}
```

### `GAME_ROUND_END`
- Sent by server once a round has ended.

#### Example
```js
{
  "t": 8,
  "d": {
    "round": {} // Round object
  }
}
```

-----
## Users
### `USER_CREATE`
- Sent by server.
- Does not necessarily mean a user has just joined the gateway.
- Send to a client if you want the client to cache this user in particular.

#### Example
```js
{
  "t": 9,
  "d": {
    "user": {} // User object
  }
}
```

### `USER_UPDATE`
- Sent by server when any property of a User object has changed (i.e. only a username)

#### Example
```js
{
  "t": 10,
  "d": {
    "user": {} // new User object
  }
}
```

-----
## Messaging
### `MESSAGE_CREATE`
- Sent by server when a message is created in a channel the receiver has access to.

#### Example
```js
{
  "t": 11,
  "d": {
    "channel_id": "<channel ID>",
    "content": "<content>",
    "timestamp": "2017-06-03T11:30:23.825Z",
    "author": {} // User object
  }
}
```