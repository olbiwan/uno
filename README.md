# [Uno](https://en.wikipedia.org/wiki/Uno_(card_game))

_This project was developed as part of the final work of the Postgraduate course at [PUCPR](https://www.pucpr.br). Inspired by the original idea of ​​[Space Invaders](https://jay-ithiel.github.io/space_invaders) developed by the AWS team, the objective is to create an online version of the game [**Uno** ](https://en.wikipedia.org/wiki/Uno_(card_game)), with a special focus on minimizing costs and avoiding **Cloud Vendor lock-in**._

## Usage

## Architecture

The architecture is based on Serverless using [**Clojure**](https://clojure.org), with external communication through a REST API, and internal communication, depending on the platform, is triggered via HTTP Trigger (in this case from [Azure Functions](https://azure.microsoft.com/en-us/products/functions)) or API Gateway (when using [AWS Lambda](https://aws.amazon.com/pt/pm/lambda)).

Microservices are organized into two main levels:

- First level (Interface and Orchestration):
This level acts as a direct interface for user requests. Your responsibility is to orchestrate calls to second-level services, integrating and composing responses.
- Second level (Application State Management):
The second level is responsible for managing and updating the state of the application. It encapsulates core logic and performs operations that change the state of the system.

```mermaid
flowchart LR
    SC -.->|Validates the informed card and updates the hand.| HPH[(Redis<br><b>hash-player-hand</b>)]
    U((begin))
    U -->|<b>POST</b> <i>/uno/player/pass-turn</i>| FD[Serverless<br><b>pass-turn-bff</b><br><i>Cloujure</i>]
    U -->|<b>POST</b> <i>/uno/security/refresh-token</i>| Z[Serverless<br><b>security</b><br><i>Cloujure</i>]
    U -->|<b>POST</b> <i>/uno/player/discard-card/:card</i>| FB[Serverless<br><b>discard-card-bff</b><br><i>Cloujure</i>]
    U -->|<b>POST</b> <i>/uno/player/draw-card</i>| FC[Serverless<br><b>draw-card-bff</b><br><i>Cloujure</i>]
    FB --> SC(Serverless<br><b>player-hand</b><br><i>Cloujure</i>)
    FD --> SA(Serverless<br><b>next-turn</b><br><i>Cloujure</i>)
    FC --> SB(Serverless<br><b>pile-card</b><br><i>Cloujure</i>)
    SA -.->|Validates the informed card and updates the last card.| HLC[(Redis<br><b>hash-last-card</b>)]
    SA -.->|Query/Update the hand.| HOH[(Redis<br><b>hash-opponent-hand</b>)]
    SB -.->|Update the pile.| HPC[(Redis<br><b>hash-pile-card</b>)]
    SA -.->|If there is no card.| SB
    SB -->|Update the hand.| SC
    SB -->|Query/Update the hand.| SA
    SC -.->|Check the opponent's move.| SA
    classDef first color:#fff,fill:green;
    classDef second color:#fff,fill:red;
    style U  color:#fff,fill:#000,stroke:#000;
    class FB,FC,FD first;
    class SA,SB,SC second;
```

**Applications:**
- **draw-card**: *Deal the cards and draw from the pile.*
- **next-turn**: *Validates the discarded card if any and returns the opponent's card.*
- **refresh-token**: *Generates the security token.*

**Database:**
<table>
    <tr>
        <th colspan="2">hash-last-card</th>
    </tr>
    <tr>
        <th>column</th>
        <th>description</th>
    </tr>
    <tr>
        <td>cards</td>
        <td>description</td>
    </tr>
    <tr>
        <td>draw</td>
        <td>description</td>
    </tr>
    <tr>
        <td>distribute</td>
        <td>description</td>
    </tr>
</table>
