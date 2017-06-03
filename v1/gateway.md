# Crude Cards - Protocol v1 _(Gateway)_
## WebSocket Gateway

1. All connections will be over `wss` (secure) - `ws` is not supported
2. Clients are only allowed to connect through `/gateway`

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
5. `GAME_PLAYERS_UPDATE`:
  - Sent by server when a player joins/leaves a game.
6. `GAME_DELETE`
  -	Sent by server once a game has been deleted or ended.
7. `GAME_ROUND_START`
  -	Sent by server when a new round in a game has started.
8. `GAME_ROUND_END`
  -	Sent by server when a new round in a game has started.
### Users
9. `USER_CREATE`
  -	Sent by server.
  - Does not necessarily mean a user has just joined the gateway.
10. `USER_UPDATE`
  - Sent when a user updates their details.

-----

## Meta Packets
### `IDENTIFY`
- Sent by client.
#### Example
```js
{
  "t": 0,
  "d": {
    "token": "abc123"
  }
}
```