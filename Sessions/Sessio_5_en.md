# Session 5

## Objectives

* Practice using nested serializers
* Complete the implementation of the game

## Exercise 1: Game state serialization

This guide explains how to create nested serializers in Django REST Framework to generate a complex JSON response for a Battleship game state. The response follows a specific schema that includes game information, player details, and board states as described in the provided JSON example.

> **⚠️ Disclaimer:**  
> **This is an example on how to do the nested serializers, adjust to your code as needed!**

### Step 1: Create the Base Structure

First, create the outer structure that matches the top-level JSON schema:

```python
class GameStateResponseSerializer(serializers.Serializer):
    # status = serializers.IntegerField(default=200)
    # message = serializers.CharField(default="OK")
    # data = serializers.SerializerMethodField()

    def get_data(self, obj):
        return {
            'gameState': GameStateSerializer(obj).data
        }
```

**Note:** Notice that in this case, since it is not a model from the application, we use `Serializer` instead of `ModelSerializer`.


### Step 2: Create the Game State Serializer

This handles the gameState object within data:

```python
class GameStateSerializer(serializers.Serializer):
    gameId = serializers.CharField(source='id')
    phase = serializers.CharField()
    turn = serializers.CharField(source='turn.nickname', allow_null=True)
    winner = serializers.CharField(source='winner.nickname', allow_null=True)
    player1 = serializers.SerializerMethodField()
    player2 = serializers.SerializerMethodField()

    def get_player1(self, obj):
        request = self.context.get('request')
        if not request:
            return None
        current_player = request.user.player
        board = Board.objects.get(game=obj, owner=current_player)
        return PlayerStateSerializer(board).data

    def get_player2(self, obj):
        request = self.context.get('request')
        if not request:
            return None
        current_player = request.user.player
        board = obj.board_set.exclude(owner=current_player).first()
        return PlayerStateSerializer(board).data if board else None
```

### Step 3: Create the Player State Serializer

This handles individual player data:

```python
class PlayerStateSerializer(serializers.Serializer):
    id = serializers.CharField(source='owner.id')
    username = serializers.CharField(source='owner.nickname')
    placedShips = serializers.SerializerMethodField()
    availableShips = serializers.SerializerMethodField()
    board = serializers.SerializerMethodField()

    def get_placedShips(self, board):
        ships = []
        for vessel in BoardVessel.objects.filter(board=board):
            ships.append({
                'type': vessel.type.id,
                'position': {
                    'row': vessel.ri,
                    'col': vessel.ci
                },
                'isVertical': vessel.rf != vessel.ri,
                'size': vessel.type.size
            })
        return ships

    def get_availableShips(self, board):
        all_vessels = Vessel.objects.all()
        placed_vessels = BoardVessel.objects.filter(board=board)
        ships = []

        for vessel in all_vessels:
            if not placed_vessels.filter(type=vessel).exists():
                ships.append({
                    'type': vessel.id,
                    'isVertical': True,
                    'size': vessel.size
                })
        return ships

    def get_board(self, board):
        size = board.game.width
        grid = [[0 for _ in range(size)] for _ in range(size)]

        # Add vessels
        for vessel in BoardVessel.objects.filter(board=board):
            value = vessel.type.id if vessel.alive else -vessel.type.id
            for i in range(min(vessel.ri, vessel.rf), max(vessel.ri, vessel.rf) + 1):
                for j in range(min(vessel.ci, vessel.cf), max(vessel.ci, vessel.cf) + 1):
                    grid[i][j] = value

        # Add shots
        for shot in Shot.objects.filter(board=board):
            if grid[shot.row][shot.col] == 0:
                grid[shot.row][shot.col] = 11  # Miss

        return grid
```

### Step 4: Update the Game Serializer

Modify your existing GameSerializer to use the new nested structure:

```python
class GameSerializer(serializers.ModelSerializer):
    game_state_response = serializers.SerializerMethodField()

    class Meta:
        model = Game
        fields = ['id', 'phase', 'turn', 'winner', 'game_state_response']

    def get_game_state_response(self, obj):
        return GameStateResponseSerializer(obj, context=self.context).data
```

### Step 5: Use in the ViewSet

Update your GameViewSet to use the serializer:

```python
class GameViewSet(viewsets.ModelViewSet):
    queryset = Game.objects.all()

    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance, context={'request': request})
        return Response(serializer.data)
```


## Exercise 2: Connecting to the Frontend

Once you have the JSON with the game state from Exercise 1, you now need to connect the frontend to this state. Modify the code of the `getGameState(gameId)` method from Session 3 so that instead of returning the example JSON, it fetches it from the backend.

Check that if you change the data in the database, the game state is updated and correctly represented on the frontend board.

## Exercise 3: Modifying the Game State

Once the frontend reflects the game state, we need to connect user actions to the corresponding backend calls. For example, during the setup phase, you’ll need to make requests to add ships, and during the game phase, to fire shots.

## Exercise 4: Refreshing the State

If you're playing single-player games, you can make the "bot" actions execute directly within the user actions that trigger them. For instance, when a user fires a shot and misses, the bot can make its moves before returning the result. In this way, if you refresh the game state after each action, everything stays in sync.

Alternatively, you can create an API action in the backend to perform the bot’s moves, and call it from a button on the frontend. The game state should also be refreshed after calling this action. This option is a bit cleaner than the previous one, as it more accurately simulates a real bot player.

Finally, you can have your frontend automatically refresh the game state at regular intervals, so you don’t need to force everything to happen before returning the state. Here's an example of how this can be done:

```vue
<template>
  <div>
    <h2>Game State</h2>
    <pre>{{ gameState }}</pre>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'
import axios from 'axios'

export default {
  name: 'GameStateRefresher',
  setup() {
    const gameState = ref(null)
    let intervalId = null

    const fetchGameState = async () => {
      try {
        const response = await axios.get('/api/games/1/') // replace with the correct endpoint
        gameState.value = response.data
      } catch (error) {
        console.error('Error fetching game state:', error)
      }
    }

    onMounted(() => {
      fetchGameState()
      intervalId = setInterval(fetchGameState, 5000) // refresh every 5 seconds
    })

    onUnmounted(() => {
      clearInterval(intervalId)
    })

    return {
      gameState
    }
  }
}
</script>
```

## Exercise 5: Listing Ongoing Games

Add a view in the frontend that displays a list of games that are waiting for players. The user should be able to select a game to join and start playing.

## Exercise 6: Leaderboard

Add a view in the frontend that displays a list of the top 5 players. For each player, their score should be shown. The score is calculated as the percentage of games won:

```
score = num_winned_games / total_games
```

The leaderboard must sort the players by score.
