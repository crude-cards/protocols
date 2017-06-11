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
  "max_players": 16,
  "max_spectators": 5,
  "public": true,
  "rules": {
    // race between score, rounds and time
    "max_score": 10, // maximum score, -1 disables the rule
    "max_rounds": 10, // maximum rounds, -1 disables the rule
    "max_round_time": 60, // maximum amount of time a round can last, -1 disables the rule
    "czar": 0 // 0 = winner, 1 = random
  },
  "current_round": {}, // Round object, can be null if game has not started
  "players": [], // array of Users
  "spectators": [], // array of Users
  "decks": [] // array of Deck IDs
}
```

### Round
```js
{
  "id": 0,
  "game": 0, // Game ID
  "start_time": "2017-06-03T11:30:23.825Z",
  "blackCard": {}, // Card object
  "czar": {}, // User object

  // following only present once the round has been completed:
  "end_time": "2017-06-03T11:30:43.825Z",
  "winner": {
    "user": {}, // user object
    "cards": [] // array of Cards used
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
  "id": "deck_id:n", // completely unique
  "text": "Beep _ boop"
}
```

### Deck
```js
{
  "id": 0,
  "owner": 0, // owner ID
  "name": "Custom Deck",
  "external": false, // true if the deck comes from Card Cast
  "cards": [] // an array of Card objects
}
```