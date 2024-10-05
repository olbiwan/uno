# [Uno](https://en.wikipedia.org/wiki/Uno_(card_game))

_**This project was developed for the Postgraduate course at [PUCPR](https://www.pucpr.br)**. Inspired by the original idea of ​​[Space Invaders](https://jay-ithiel.github.io/space_invaders) developed by the AWS team, the objective is to create an online version of the game [**Uno** ](https://en.wikipedia.org/wiki/Uno_(card_game)), with a special focus on minimizing costs and avoiding **Cloud Vendor lock-in**._

## Usage

## Architecture

The architecture is based on Serverless using [**Clojure**](https://clojure.org), with external communication through a REST API, and internal communication, depending on the platform, is triggered via HTTP Trigger (in this case from [Azure Functions](https://azure.microsoft.com/en-us/products/functions)) or API Gateway (when using [AWS Lambda](https://aws.amazon.com/pt/pm/lambda)).

Microservices are organized into two main levels:

- Back-end for Front-end (BFF):
This level acts as a direct interface for user requests. Your responsibility is to orchestrate calls to second-level services, integrating and composing responses.
- Second level (Application State Management):
The second level is responsible for managing and updating the state of the application. It encapsulates core logic and performs operations that change the state of the system.

### Services Available

<details>
  <summary><b>Deliver cards and draw from the pile.</b> <i>(click to see)</i></summary>

```mermaid
flowchart TD
    U((begin))
    U -->|<b>POST</b> <i>/uno/security/refresh-token</i>| Z[<b>security</b>]
    U -->|<b>POST</b> <i>/uno/player/draw-card</i>| FC[<b>draw-card-bff</b>]
    FC -->|Returns the player's cards and last card.| SB(<b>pile-card</b>)
    SA(<b>next-turn</b>) -.->|Query/update the last card.| HLC[(<b>last-card</b>)]
    SA -.->|Update the hand.| HOH[(<b>opponent-hand</b>)]
    SB -.->|Query/Update the pile.| HPC[(<b>pile-card</b>)]
    SB -->|Update the hand.| SC
    SB -->|Update the hand.| SA
    SC(<b>player-hand</b>) -.->|Update the hand.| HPH[(<b>player-hand</b>)]
    classDef first color:#fff,fill:green;
    classDef second color:#fff,fill:red;
    style U  color:#fff,fill:#000,stroke:#000;
    class FC first;
    class SA,SB,SC second;
```

</details>

<details>
  <summary><b>Discard the player's card.</b> <i>(click to see)</i></summary>

```mermaid
flowchart TD
    U -->|<b>POST</b> <i>/uno/security/refresh-token</i>| Z[Serverless<br><b>security</b><br><i>Cloujure</i>]
    U((begin))
    U -->|<b>POST</b> <i>/uno/player/discard-card/:card</i>| FB[<b>discard-card-bff</b>]
    FB --> SC(<b>player-hand</b>)
    SA(<b>next-turn</b>) -.->|Validates the informed card and updates the last card.| HLC[(<b>last-card</b>)]
    SA -.->|Query/Update the hand.| HOH[(<b>opponent-hand</b>)]
    SB(<b>pile-card</b>)  -.->|Update the pile.| HPC[(<b>pile-card</b>)]
    SC -.->|Validates the informed card and updates the hand.| HPH[(<b>player-hand</b>)]
    SA -->|If there is no card.| SB
    SC -->|Check the opponent's move.| SA
    classDef first color:#fff,fill:green;
    classDef second color:#fff,fill:red;
    style U  color:#fff,fill:#000,stroke:#000;
    class FB,FC,FD first;
    class SA,SB,SC second;
```

</details>

<details open>
  <summary><b>Pass the turn to the opponent.</b> <i>(click to see)</i></summary>

```mermaid
flowchart TD
    U((begin)) -->|<b>POST</b> <i>/uno/security/refresh-token</i>| Z[<b>security</b>]
    U -->|<b>POST</b> <i>/uno/player/pass-turn</i>| FD[<b>pass-turn-bff</b>]
    FD --> SA(<b>next-turn</b>)
    SA -.->|Update the last card.| HLC[(<b>last-card</b>)]
    SA -.->|Query/Update the hand.| HOH[(<b>opponent-hand</b>)]
    SB(<b>pile-card</b>) -.->|Update the pile.| HPC[(<b>pile-card</b>)]
    SA -.->|If there is no card.| SB
    classDef first color:#fff,fill:green;
    classDef second color:#fff,fill:red;
    style U  color:#fff,fill:#000,stroke:#000;
    class FB,FC,FD first;
    class SA,SB,SC second;
```

</details>

**Applications:**
- **draw-card**: *Deal the cards and draw from the pile.*
- **next-turn**: *Validates the discarded card if any and returns the opponent's card.*
- **refresh-token**: *Generates the security token.*
