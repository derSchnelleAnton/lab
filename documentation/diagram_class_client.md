classDiagram
  %% Eigene Client-Typen
  class PlayerClient {
    -ClientModel model
    -PlayerInput input
    -RestClient restClient
    -GameConnection connection
    +start()
    +connectToGameServer()
    +setInput(i : PlayerInput)
    +toggleAiMode(ai : boolean)
  }

  class FrontEnd {
    <<package>>
    Front-End der Web-App, wird nicht weiter modelliert
    +render(model : ClientModel)
    +toggleAiMode(ai : boolean)
  }

  class PlayerInput {
    <<interface>>
    +buildShift(direction : Direction, index : int, rotation : Rotation) ShiftAction
    +buildMove(target : Position) MoveAction
  }

  class HumanInput
  class ArtificialInput

  class GameConnection {
    -WebSocket webSocket
    +send(a : Action)
    +onMessage(m : Message)
  }

  class ClientModel {
    -Board boardMirror
    -GameState state
    -ActionType currentAction
    +apply(u : StateUpdateMessage)
    +validateActionLocally(a : Action) boolean
  }

  class RestClient {
    <<interface>>
    +post(path : String, body : String) String
    +get(path : String) String
    +delete(path : String)
  }

  %% Externe Typen aus anderen Modulen
  class Board
  class Game
  class Action
  class ShiftAction
  class MoveAction
  class Message
  class StateUpdateMessage
  class Position
  class Direction
  class Rotation
  class GameState
  class ActionType
  class WebSocket

  PlayerClient "1" *-- "1" FrontEnd
  PlayerClient "1" o-- "1" PlayerInput : aktive Strategie
  PlayerClient "1" *-- "1" GameConnection
  PlayerClient "1" *-- "1" ClientModel
  PlayerClient "1" --> "1" RestClient : list / join
  FrontEnd --> ClientModel : rendert
  FrontEnd ..> PlayerClient : schaltet KI-Modus
  HumanInput ..|> PlayerInput
  ArtificialInput ..|> PlayerInput
  ArtificialInput ..> ClientModel : liest Spiegel
  PlayerInput ..> ShiftAction : erzeugt
  PlayerInput ..> MoveAction : erzeugt
  GameConnection --> WebSocket : kapselt
  GameConnection ..> Message : sendet / empfaengt
  GameConnection ..> Action : versendet
  ClientModel ..> Board : Spiegel
  ClientModel ..> Game : Regel-Logik
  ClientModel ..> StateUpdateMessage : verarbeitet