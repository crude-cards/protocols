# Crude Cards - Protocol v1 _(Draft)_
## Data Structures
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
  },
  "currentRound": {}, // Round object, can be null if game has not started
  "players": [], // array of Users
  "spectators": [], // array of Users
  "decks": [] // array of Decks used in the game
}
```

### Round
```js
{
  "id": 0,
  "game": 0, // Game ID
  "startTime": "2017-06-03T11:30:23.825Z",
  "blackCard": {}, // card object
  "czar": {}, // user object

  // following only present once the round has been completed:
  "endTime": "2017-06-03T11:30:43.825Z",
  "winner": {
    "user": {}, // user object
    "cards": [] // array of cards used
  }
}
```

### Message
```js
{
  "author": 0, // User ID
  "time": "2017-06-03T11:30:43.825Z",
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