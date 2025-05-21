# Sessió 5

## Objectius

- Practicar en l'ús de serialitzadors imbricats
- Finalitzar la implementació del joc

## Exercici 1: Serialització de l'estat de la partida

Aquesta guia explica com crear serialitzadors imbricats amb Django REST Framework per generar una resposta JSON complexa per a l'estat d'una partida. La resposta segueix l'esquema que es va donar com a exemple en el frontend, incloent informació de la partida, detalls dels jugadors i els estats dels taulers segons l'exemple JSON proporcionat.

> **⚠️ Avís:**
> **Aquest és un exemple de com fer els serialitzadors imbricats; adapta'l al teu codi segons sigui necessari!**

### Pas 1: Crea l’Estructura Base

Primer, crea l’estructura externa que coincideixi amb l’esquema JSON de nivell superior:

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

**Nota:** Fixa't que en aquest cas, al no tractar-se d'un model de l'aplicació, utilitzem Serializer en comptes de ModelSerializer.

### Pas 2: Crea el Serialitzador de l’Estat del Joc

Aquest gestiona l’objecte `gameState` dins de `data`:

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

### Pas 3: Crea el Serialitzador de l’Estat del Jugador

Aquest gestiona les dades individuals dels jugadors:

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

        # Afegeix els vaixells
        for vessel in BoardVessel.objects.filter(board=board):
            value = vessel.type.id si vessel.alive else -vessel.type.id
            for i in range(min(vessel.ri, vessel.rf), max(vessel.ri, vessel.rf) + 1):
                for j in range(min(vessel.ci, vessel.cf), max(vessel.ci, vessel.cf) + 1):
                    grid[i][j] = value

        # Afegeix els trets
        for shot in Shot.objects.filter(board=board):
            if grid[shot.row][shot.col] == 0:
                grid[shot.row][shot.col] = 11  # Fallat

        return grid
```

### Pas 4: Actualitza el Serialitzador de Partides

Modifica el teu `GameSerializer` existent per utilitzar l’estructura imbricada nova:

```python
class GameSerializer(serializers.ModelSerializer):
    game_state_response = serializers.SerializerMethodField()

    class Meta:
        model = Game
        fields = ['id', 'phase', 'turn', 'winner', 'game_state_response']

    def get_game_state_response(self, obj):
        return GameStateResponseSerializer(obj, context=self.context).data
```

### Pas 5: Utilitza-ho al ViewSet

Actualitza el teu `GameViewSet` per utilitzar el serialitzador:

```python
class GameViewSet(viewsets.ModelViewSet):
    queryset = Game.objects.all()

    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance, context={'request': request})
        return Response(serializer.data)
```

## Exercici 2: Connectar amb el frontend

Un cop teniu el JSON amb l'estat d'una partid de l'exercici 1, ara cal que connecteu el frontend amb aquest estat. Modifica el codi del mètode ```getGameState(gameId)``` de la sessió 3 per tal que en comptes de retornar el JSON d'exemple, ara l'obtingui del backend. 

Comproveu que si modifiqueu les dades de la base de dades l'estat de la partida es veu alterat i representat correctament en el tauler del frontend.

## Exercici 3: Modificació de l'estat de la partida

Un cop el frontend reflecteix l'estat de la partida, ara ens cal connectar les accions de l'usuari amb les crides corresponents al backend. Per exemple, en la fase de posicionament caldrà fer les crides per afegir els vaixells, i en la fase de joc per disparar. 


## Exercici 4: Refrescant l'estat

Si feu partides d'un sol jugador, podeu fer que les accions del "bot" s'executin directament dins de les accions de l'usuari que les genera. Per exemple, quan un usuari fa una tirada que falla, el bot pot fer les seves tirades abans de retornar el resultat. D'aquesta forma, si després de cada acció refresqueu l'estat de la partida, ja us quedarà tot connectat.

També podeu crear una acció a l'API del backend per a que faci les accions del bot, i feu la crida a partir d'un botó del frontend. El refresc de l'estat es farà també després de fer la crida a l'acció. Aquesta opció és una mica més neta que l'anterior, ja que simula realment un jugador bot.

Finalment, podeu fer que el vostre frontend es refresqui automàticament cada cert temps, amb el que no cal que forceu a que tot es faci abans de retornar l'estat. Aquí teniu un exemple de com es pot fer:

```vue
<template>
  <div>
    <h2>Estat del Joc</h2>
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
        const response = await axios.get('/api/games/1/') // substitueix amb l’endpoint correcte
        gameState.value = response.data
      } catch (error) {
        console.error('Error obtenint l’estat del joc:', error)
      }
    }

    onMounted(() => {
      fetchGameState()
      intervalId = setInterval(fetchGameState, 5000) // refresca cada 5 segons
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

## Exercici 5: Llistat de partides en joc

Afegeix una vista al frontend que mostri la llista de partides que estan esperant jugadors. L'usuari ha de poder triar una partida per afegir-s'hi i començar a jugar.


## Exercici 6: Tauler de lideratge

Afegeix una vista al frontend que mostri la llista dels 5 millors jugadors. Per a cada jugador cal que es mostri la seva puntuació. La puntuació es calcula com el percentatge de partides guanyades:

```
score = num_winned_games / total_games
```

El tauler ha d'ordenar els jugadors per puntuació.




