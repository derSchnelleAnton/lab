sequenceDiagram
    autonumber
    participant A as Aktiver Spieler
    participant S as Spielserver
    participant B as Alle Spieler

    loop bis Partieende
        Note over A,S: Phase WAITING_FOR_PUSH
        A->>S: ROTATE_SPARE_TILE (optional)<br/>90 Grad im Uhrzeigersinn
        A->>S: PUSH_TILE<br/>ungerader innerer Index, Richtung
        alt Schub erlaubt
            S-->>B: GAME_STATE_UPDATE (Broadcast)<br/>WAITING_FOR_MOVE, serverseitig berechnete reachablePositions
        else unerlaubt / nicht am Zug
            S-->>A: ERROR (Unicast an Verursacher)<br/>INVALID_PUSH bzw. NOT_YOUR_TURN<br/>Zustand unverändert, Wiederholung erlaubt
        end

        Note over A,S: Phase WAITING_FOR_MOVE
        alt Zug mit Bewegung
            A->>S: MOVE_FIGURE (targetCoordinates)
            opt Zielfeld enthält aktuellen Schatz
                S-->>B: TREASURE_FOUND (Broadcast)
                S-->>A: NEXT_TREASURE (Unicast, nur an Berechtigten)
            end
        else Zug ohne Bewegung
            A->>S: PASS_MOVE
        end
        S-->>B: GAME_STATE_UPDATE (Broadcast)<br/>Full-State, nächster Spieler ist aktiv
    end

    Note over B: Siegbedingung: alle Schätze gefunden<br/>und Rückkehr zur homePosition
    S-->>B: GAME_OVER (Broadcast)<br/>Sieger, Rangliste, Kennzahlen