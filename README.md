# [Hello World Open](http://helloworldopen.fi) Technical Specifications

## The Game

Each team should provide a program (called a "bot") that works as a client for the game server. The game server launches matches of a Pong-like game between competitors. The bot controls a paddle on the game that should prevent ball from exiting the game area through the end of the area guarded by the paddle. When the ball exits through opponent's end, the bot wins a game. In a tournament between two bots multiple games can be played and the bot that receives more points wins the tournament.

![Game visualization](hwo-techspec/raw/master/game-snapshot.png "Game visualization")

Outside tournaments bots can practice against test opponents on a test server. These opponents are test bots provided specifically to train the competition bots. They are not the bots implemented by other teams. Only during tournaments (qualifiers, semi-finals and finals) the bots compete against bots written by other competing teams.

## Bot requirements

Each bot can send up to 20 messages per 2 seconds to the game server. Flooding the game server with more messages can result in kicking the bot out of the game and game won by the opposing bot.

Each bot should be shipped with three scripts:

- A script to build the bot:<br/>
`./build.sh`

- A script to run the bot:<br/>
`./start.sh <bot-name> <host> <port>`<br/>
This script has to be non-blocking.

- A script to stop the bot:<br/>
`./stop.sh`

## Protocol between client and server

### Game protocol

- The game protocol consists of JSON messages over TCP/IP
- Messages are single JSON objects on one line, separated by newlines (`\n`)
- One game message contains fields of `msgType` and `data`
- The client starts the game with a JOIN message where
  - the msgType is 'join'
  - the data is the name of the client

~~~ json
     {"msgType":"join","data":"JohnMcEnroe"}
~~~

- Alternative way to join a game is to request a duel. Once there are two
  matching requests the game is started between the two.

~~~ json
    {"msgType":"requestDuel","data":["mybotname", "requested duel partner's name"]}
~~~

- Once the client has joined a match the server sends a 'joined' message 
  - the msgType is 'joined'
  - the data is the URL of the game visualization

~~~ json
     {"msgType":"joined","data":"http://boris.helloworldopen.fi/visualize.html#game_id"}
~~~


- The client waits until server starts a game
- The game start message contains a list of player names where
  - the msgType is 'gameStarted'
  - the data is a list of clients competing in the game

~~~ json
     {"msgType":"gameStarted","data":["JohnMcEnroe", "BorisBecker"]}
~~~

- During the game, client and server communicate with game-specific messages (see 'Game protocol during a match' for more information)
- The game ends with a message containing the name of the winning bot where
  - the msgType is 'gameIsOver'
  - the data contains the name of the winning client

~~~ json
     {"msgType":"gameIsOver","data":"JohnMcEnroe"}
~~~

- In a tournament match, multiple games are played. A single TCP connection must be maintained between games.
- If a client disconnects during tournament, there's no way to reconnect.
- Prepare your client for non-recognized message types!

### Game protocol during a match

- Server sends messages with the game state to the client during the game,
- The message defines the state of the game (timestamp, paddle positions, ball position, sizes).
- The client is always on the left side of the board.
- Balls and paddles alike will bounce from board edges.
- Example board status message:

~~~ json
    { "msgType":"gameIsOn",
      "data": { "time":1336219278079,
                "left":{"y":186.0,"playerName":"JohnMcEnroe"},
                "right":{"y":310.0,"playerName":"BorisBecker"},
                "ball":{"pos":{"x":291.0,"y":82.0}},
                "conf":{"maxWidth":640,"maxHeight":480,"paddleHeight":50,"paddleWidth":10,"ballRadius":5,"tickInterval":15}}}
~~~

- The client can send a paddle control message, indicating paddle direction and speed at any time during the game.
- The paddle keeps moving with the specified speed until a new control message is sent.
- The control message defines the paddle speed and direction as a floating point value on the y-axis. The provided value must be in the range of [-1 .. 1] where -1.0 indicates full speed up and 1.0 full speed down. A control message with the speed of 0.0 immediately stops the paddle.
- Example control message:

~~~ json
    {"msgType":"changeDir","data":1.0}
~~~

## Testing game server

The testing game server pits a practice bot against any bot that joins. The practice bots might become more challenging as the competition progresses. Teams get to play their bots against each other only in tournament play.

The testing game server is hosted at boris.helloworldopen.fi port 9090.

## Visualization

The matches are visualized once they have been created. The visualization uses the same exact data that the client recieves. Each player recieves a unique id that can be used to observe the game in progress.

The visualization urls look like this http://boris.helloworldopen.fi:8080/visualize.html#game_id
