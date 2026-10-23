sequenceDiagram
    autonumber
    participant C as Client
    participant S as Spielserver

    C->>S: WebSocket-Handshake ws://host:port/game
    S-->>C: SERVER_INFO (Unicast)<br/>serverVersion, serverTime, motd

    C->>S: CONNECT (Benutzername)<br/>innerhalb von 10 Sekunden
    alt Platz frei, Name frei, Token gültig
        S-->>C: CONNECT_ACK (Unicast)<br/>playerId, Farbe, isAdmin, identifierToken
        S-->>C: LOBBY_STATE (Broadcast an alle)<br/>Spielerliste, canStart
    else Lobby voll / Name vergeben / Spiel läuft ohne Token
        S-->>C: ERROR (Unicast)<br/>LOBBY_FULL, USERNAME_TAKEN bzw. GAME_ALREADY_STARTED
    end

    C->>S: START_GAME (nur Admin, min. 2 Spieler)<br/>boardSize u. a.
    S-->>C: GAME_STARTED (Broadcast)<br/>Ersatzkachel, gameEndTime
    S-->>C: GAME_STATE_UPDATE (Broadcast)<br/>vollständiger Spielzustand, keine Deltas
    S-->>C: NEXT_TREASURE (Unicast, nur an berechtigten Spieler)<br/>erste geheime Schatzkarte

    Note over C,S: Verbindungsabbruch während der Partie
    S-->>C: PLAYER_UPDATED (Broadcast)<br/>isConnected: false

    alt Reconnect innerhalb von 30 Sekunden
        C->>S: CONNECT (Benutzername, identifierToken)
        S-->>C: CONNECT_ACK (Unicast)
        S-->>C: GAME_STATE_UPDATE (Unicast, Full-State)
        S-->>C: NEXT_TREASURE (Unicast)
    else Fenster abgelaufen
        S-->>C: PLAYER_DISCONNECTED (Broadcast)<br/>reason: TIMEOUT, Figur wird entfernt
    end