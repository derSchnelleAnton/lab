sequenceDiagram
    autonumber
    participant C as Client
    participant G as Spielserver
    participant V as Verzeichnisserver

    Note over G,V: Hochfahren des Spielservers
    G->>V: POST /api/servers (X-API-Key)<br/>name, host, port, maxPlayers
    alt gültiger Key
        V-->>G: 201 serverId, heartbeatIntervalSeconds: 10
    else Key fehlt/ungültig/mehrfach
        V-->>G: 401 INVALID_API_KEY (vor Body-Parsing)
    else Body nicht lesbar oder Validierung fehlgeschlagen
        V-->>G: 400 INVALID_CONFIG
    end

    loop alle 10 Sekunden
        G->>V: PUT /api/servers/{id}/heartbeat (X-API-Key)<br/>status, currentPlayers, maxPlayers
        alt eigenes Team
            V-->>G: 200 acknowledged: true
        else fremdes Team
            V-->>G: 403 SERVER_ACCESS_DENIED
        else Server nicht (mehr) registriert
            V-->>G: 404 SERVER_NOT_FOUND
            Note over G,V: Spielserver registriert sich neu (POST)
        end
    end

    Note over V: Letztes Lebenszeichen älter als 30 s<br/>→ Server wird entfernt (Prüfung im 5-s-Takt)

    C->>V: GET /api/servers (?status=LOBBY|RUNNING|UNKNOWN)
    V-->>C: 200 Liste mit serverId, host, port, Auslastung, Status
    Note over C,G: host und port ergeben ws://host:port/game
    C->>G: WebSocket-Verbindung /game (siehe asyncapi.yaml)

    Note over G,V: Herunterfahren des Spielservers
    G->>V: DELETE /api/servers/{id} (X-API-Key)
    alt eigenes Team
        V-->>G: 204 kein Body
    else fremdes Team
        V-->>G: 403 SERVER_ACCESS_DENIED
    else erneut abgemeldet / nicht registriert
        V-->>G: 404 SERVER_NOT_FOUND
    end