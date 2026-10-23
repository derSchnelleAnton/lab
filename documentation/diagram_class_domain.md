classDiagram
  class Game {
    -Board board
    -List~Player~ players
    -Player currentPlayer
    -Turn currentTurn
    -GameState state
    +start()
    +executeAction(a : Action) boolean
    +checkWinCondition(p : Player) boolean
    +nextTurn()
  }

  class Turn {
    -Player player
    -Deque~ActionType~ pendingActions
    -List~Action~ executedActions
    +current() ActionType
    +matches(a : Action) boolean
    +consume(a : Action)
    +grant(t : ActionType)
    +isFinished() boolean
  }

  class Board {
    -Tile~7x7~ grid
    -Tile shiftTile
    +validateShift(a : ShiftAction) boolean
    +applyShift(a : ShiftAction)
    +validateMove(a : MoveAction) boolean
    +applyMove(a : MoveAction)
    +reachableTiles() Set~Position~
  }

  class Tile {
    -int pathBits
    -Rotation rotation
    -List~Collectable~ collectables
    -List~Player~ players
    +openSides() Set~Direction~
    +rotate(r : Rotation)
  }

  class Player {
    -String name
    -int score
    -Deque~Treasure~ remainingTreasures
    -Treasure currentTarget
    -Position position
    +collect(c : Collectable)
  }

  class Position {
    +int row
    +int col
  }

  class Collectable {
    <<interface>>
    +onCollect(p : Player)
  }

  class Treasure {
    -TreasureType type
  }

  class Bonus {
    -BonusType type
  }

  class Action {
    <<interface>>
    +ActionType type
  }

  class ShiftAction {
    +Direction direction
    +int index
    +Rotation rotation
  }

  class MoveAction {
    +Position target
  }

  class GameState {
    <<enumeration>>
    LOBBY
    RUNNING
    FINISHED
  }

  class ActionType {
    <<enumeration>>
    SHIFT
    MOVE
  }

  class Rotation {
    <<enumeration>>
    DEG_0
    DEG_90
    DEG_180
    DEG_270
  }

  class Direction {
    <<enumeration>>
    UP
    DOWN
    LEFT
    RIGHT
  }

  class TreasureType {
    <<enumeration>>
    24 festgelegte Schaetze
  }

  class BonusType {
    <<enumeration>>
    DOUBLE_MOVE
  }

  Game "1" *-- "1" Board
  Game "1" *-- "2..4" Player
  Game "1" *-- "1" Turn
  Game --> GameState : state
  Game ..> Action : verarbeitet
  Turn "1" --> "1" Player : player
  Turn --> ActionType : wartet auf
  Turn --> "0..*" Action : verlauf
  Bonus ..> Turn : gewaehrt Aktion
  Board "1" *-- "49" Tile : grid
  Board "1" o-- "1" Tile : shiftTile
  Tile "1" o-- "0..4" Player : traegt
  Tile "1" o-- "0..2" Collectable
  Treasure ..|> Collectable
  Bonus ..|> Collectable
  ShiftAction ..|> Action
  MoveAction ..|> Action
  Player "1" --> "1" Position : position
  Player "0..1" --> "1" Treasure : currentTarget
  MoveAction --> Position : target
  ShiftAction --> Direction : direction
  Tile --> Rotation : rotation
  Treasure --> TreasureType : type
  Bonus --> BonusType : type