## Structures
> Structures for data types used by Crude Cards

See also: [REST API](rest.md) and [Gateway](gateway.md)

### User
```js
{
  "id": 0,
  "username": "Bob"
}
```

### Game
```js
{
  "id": 0,
  "owner": 0, // User ID
  "name": "Test Game",
  "maxPlayers": 16,
  "maxSpectators": 5,
  "public": true,
  "rules": {
    "maxScore": 10,
    "maxRounds": 10, // race condition between score and rounds
    "czar": 0 // 0 = winner, 1 = random
  }
  "players": [], // array of Users
  "spectators": [], // array of Users
  "decks": [] // array of Decks used in the game
}
```

### Message
```js
{
  "author": 0, // User ID
  "timestamp": 1496433919,
  "content": "message content!",
  "channel": 0, // -1 is global, any other number should be a Game ID
}
```

### Card
```js
{
  "type": 0, // 0 = black, 1 = white
  "id": 0, // unique to a deck
  "content": "Beep _ boop"
}
```

### Deck
```js
{
  "id": 0,
  "owner": 0, // owner ID
  "name": "Custom Deck",
  "cards": [] // an array of Card objects
}
```