flowchart LR
    subgraph ARCH["High-Level-Architektur: Das verrückte Labyrinth"]
        direction LR
        S["Spielserver × n<br/>(je Team)"]
        V["Verzeichnisserver × 1"]
        C["Clients × n<br/>(Spieler)"]

        S -- "POST Serverinformationen" --> V
        V -- "GET gebündelte Serverliste" --> C
        S <-. "WebSocket: Spielzustand" .-> C
    end