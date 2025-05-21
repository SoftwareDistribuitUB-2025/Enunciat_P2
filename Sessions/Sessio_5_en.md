# Creating Nested Serializers for the Game State JSON Response

## Overview

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
