# [Uno](https://en.wikipedia.org/wiki/Uno_(card_game))

_**This project was developed for the Postgraduate course at [PUCPR](https://www.pucpr.br)**. Inspired by the original idea of ​​[Space Invaders](https://jay-ithiel.github.io/space_invaders) developed by the AWS team, the objective is to create an online version of the game [**Uno** ](https://en.wikipedia.org/wiki/Uno_(card_game)), with a special focus on minimizing costs and avoiding **Cloud Vendor lock-in**._

## Usage

## Architecture

A arquitetura é baseada em um modelo serverless usando [**Clojure**](https://clojure.org), com comunicação externa feita por meio de uma API REST. A comunicação interna varia dependendo da plataforma: usa HTTP Triggers no caso do [Azure Functions](https://azure.microsoft.com/en-us/products/functions), ou API Gateway ao utilizar [AWS Lambda](https://aws.amazon.com/pt/pm/lambda).

**Microservices are organized into two main levels**:

- **Back-end for Front-end (BFF)**:
_This level acts as a direct interface for user requests. Your responsibility is to orchestrate calls to second-level services, integrating and composing responses._
- **Second level (Application State Management)**:
_The second level is responsible for managing the application state and encapsulating the core logic._

### Services Available

<details>
  <summary><b>Deliver cards and draw from the pile.</b> <i>(click to see)</i></summary>

```mermaid
flowchart TD
    U((begin)) -->|<b>POST</b> <i>/uno/security/refresh-token</i>| S[security]
    U -->|<b>POST</b> <i>/uno/player/draw-card</i>| DCB[<b>draw-card-bff</b>]
    DCB -->|Returns the player's cards and last card.| SPC(pile-card)
    SNT(next-turn) -.->|Query/update the last card.| HLC[(last-card)]
    SPC -->|Update the hand.| SPH(player-hand)
    SPC -->|Update the hand.| SNT
    SNT -.->|Update the hand.| HOH[(opponent-hand)]
    SPC -.->|Query/Update the pile.| HPC[(pile-card)]
    SPH -.->|Update the hand.| HPH[(player-hand)]
    style DCB color:#fff,fill:green;
    style U color:#fff,fill:#000,stroke:#000;
    classDef second color:#fff,fill:red;
    class SNT,SPC,SPH second;
```

</details>

<details>
  <summary><b>Discard the player's card.</b> <i>(click to see)</i></summary>

```mermaid
flowchart TD
    U((begin)) -->|<b>POST</b> <i>/uno/security/refresh-token</i>| Z[security]
    U -->|<b>POST</b> <i>/uno/player/discard-card/:card</i>| FB[<b>discard-card-bff</b>]
    FB --> SC(player-hand)
    SA(next-turn) -.->|Validates the informed card and updates the last card.| HLC[(last-card)]
    SA -.->|Query/Update the hand.| HOH[(opponent-hand)]
    SB(pile-card)  -.->|Update the pile.| HPC[(pile-card)]
    SC -.->|Validates the informed card and updates the hand.| HPH[(player-hand)]
    SA -->|If there is no card.| SB
    SC -->|Check the opponent's move.| SA
    classDef first color:#fff,fill:green;
    classDef second color:#fff,fill:red;
    style U  color:#fff,fill:#000,stroke:#000;
    class FB first;
    class SA,SB,SC second;
```

</details>

<details>
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
