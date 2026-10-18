classDiagram
  %% Eigene Spielserver-Typen
  class GameServer {
    -Map~String, Game~ games
    -Map~Session, PlayerSession~ sessions
    -RestClient restClient
    -WebSocketServer webSocket
    +start()
    +registerAtDirectory()
    +onMessage(s : PlayerSession, m : Message)
    +broadcast(m : Message)
  }

  class PlayerSession {
    -Session connection
    -Player player
    +send(m : Message)
    +handle(m : Message)
  }

  class Session {
    <<interface>>
    +send(m : Message)
  }

  class RestClient {
    <<interface>>
    +post(path : String, body : String) String
    +get(path : String) String
    +delete(path : String)
  }

  class WebSocketServer {
    <<interface>>
    +start(port : int)
  }

  %% Externe Typen aus anderen Modulen
  class Game
  class Player
  class Message

  GameServer "1" *-- "1..4" PlayerSession
  GameServer "1" o-- "1..*" Game : hostet
  GameServer "1" --> "1" RestClient : register / heartbeat / deregister
  GameServer "1" --> "1" WebSocketServer : referenziert Endpunkt
  GameServer ..> PlayerSession : broadcastet
  GameServer ..> Message : sendet / empfaengt
  PlayerSession "1" --> "1" Session : kapselt
  PlayerSession ..> GameServer : ruft onMessage()
  PlayerSession ..> Message : sendet
  WebSocketServer --> Session : verwaltet Verbindungen