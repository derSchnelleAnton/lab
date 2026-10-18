stateDiagram-v2
    [*] --> Nicht_Registriert
    Nicht_Registriert --> UNKNOWN: POST /api/servers (201)<br/>Erstes Lebenszeichen, currentPlayers = 0
    state "Registriert" as Reg {
        UNKNOWN --> LOBBY: Heartbeat status=LOBBY
        UNKNOWN --> RUNNING: Heartbeat status=RUNNING
        LOBBY --> RUNNING: Heartbeat status=RUNNING
        RUNNING --> LOBBY: Heartbeat status=LOBBY
        LOBBY --> UNKNOWN: Heartbeat status=UNKNOWN
        RUNNING --> UNKNOWN: Heartbeat status=UNKNOWN
    }
    UNKNOWN --> Nicht_Registriert: DELETE (204) oder<br/>Timeout > 30 s ohne Heartbeat
    LOBBY --> Nicht_Registriert: DELETE (204) oder<br/>Timeout > 30 s ohne Heartbeat
    RUNNING --> Nicht_Registriert: DELETE (204) oder<br/>Timeout > 30 s ohne Heartbeat
    Nicht_Registriert --> [*]: Neustart des Verzeichnisservers<br/>löscht das gesamte Verzeichnis