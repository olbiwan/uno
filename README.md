![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?logo=terraform&logoColor=white)
![Clojure](https://img.shields.io/badge/Clojure-%23Clojure.svg?logo=Clojure)
![Postman](https://img.shields.io/badge/Postman-FF6C37?logo=postman&logoColor=white)
![React](https://img.shields.io/badge/React-%2320232a.svg?logo=react&logoColor=%2361DAFB)

# [Uno](https://en.wikipedia.org/wiki/Uno_(card_game))

_**This project was developed for the Postgraduate course at [PUCPR](https://www.pucpr.br)**. Inspired by the original idea of ​​[Space Invaders](https://jay-ithiel.github.io/space_invaders) developed by the AWS team, the objective is to create an online version of the game [**Uno**](https://en.wikipedia.org/wiki/Uno_(card_game)), with a special focus on minimizing costs and avoiding **Cloud Vendor lock-In**._

## Usage

## Architecture

The architecture is based on a **serverless model** using [**Clojure**](https://clojure.org), with external communication handled via a `REST API`. Internal communication varies depending on the platform: it uses `HTTP Triggers` in the case of [Azure Functions](https://azure.microsoft.com/en-us/products/functions) or `API Gateway` when using [AWS Lambda](https://aws.amazon.com/lambda).

**Microservices are organized into two main levels**:

- **Back-end for Front-end (BFF)**:
This level acts as a direct interface for user requests. Your responsibility is to orchestrate calls to second-level services, integrating and composing responses.
- **Second level (Application State Management)**:
The second level is responsible for managing the application state and encapsulating the core logic.

### Services Available

<details>
  <summary><code>POST</code> <code><b>/uno/player/discard-card/:card</b></code> <i>(discard the player's card)</i></summary>

```mermaid
flowchart LR
    U((begin)) --->|<b>POST</b> <i>/uno/security/refresh-token</i>| S[security-bff]
    S -.-> DS[(security)]
    U --->|<b>POST</b> <i>/uno/player/discard-card/:card</i>| DCB[<b>discard-card-bff</b>]
    DCB --> SC(player)
    SA(dealer) -.->|Validates the informed card and updates the last card.| DLC[(last-card)]
    SA -.->|Query/Update the hand.| DOH[(opponent-hand)]
    SB(pile-card)  -.->|Update the pile.| DPC[(pile-card)]
    SC -.->|Validates the informed card and updates the hand.| DPH[(player-hand)]
    SA -->|If there is no card.| SB
    SC -->|Check the opponent's move.| SA    
    style U  color:#fff,fill:#000,stroke:#000;
    classDef bff color:#fff,fill:green;
    class S,DCB bff;
    classDef second color:#fff,fill:red;
    class SA,SB,SC second;
```
</details>

<details>
  <summary><code>POST</code> <code><b>/uno/player/draw-card</b></code> <i>(deal and draw cards from the pile)</i></summary>

#### Parameters

> | Name          |  Type  | Description       |
> |---------------|--------|-------------------|
> | Authorization | Header | *Security token.* |

#### Responses

> | HTTP Code | Description                                 |
> |-----------|---------------------------------------------|
> | `200`     | *Returns the player's cards and last card.* |
> | `401`     | *Invalid authentication.*                   |
> | `409`     | *Invalid play.*                             |

```mermaid
flowchart LR
    U((begin)) --->|<b>POST</b> <i>/uno/security/refresh-token</i>| S[security-bff]
    S -.-> DS[(security)]
    U --->|<b>POST</b> <i>/uno/player/draw-card</i>| DCB[draw-card-bff]
    DCB --> SPC(pile-card)
    SD(dealer) -.->|Query/update the last card.| DLC[(last-card)]
    SPC --> SP(player)
    SPC --> SD
    SD -.->|Update the hand.| DOH[(opponent-hand)]
    SPC -.->|Query/Update the pile.| DPC[(pile-card)]
    SP -.->|Update the hand.| DPH[(player-hand)]
    style U color:#fff,fill:#000,stroke:#000;
    classDef bff color:#fff,fill:green;
    class S,DCB bff;
    classDef second color:#fff,fill:red;
    class SD,SP,SPC second;
```

#### Example cURL

> ```javascript
>  curl --request POST '{{uno}}/uno/player/draw-card' \
>  --header 'Authorization: {{authorization}}'
> ```

</details>

<details>
  <summary><code>POST</code> <code><b>/uno/player/pass-turn</b></code> <i>(pass the turn to the opponent)</i></summary>

```mermaid
flowchart LR
    U((begin)) --->|<b>POST</b> <i>/uno/security/refresh-token</i>| S[security]
    U --->|<b>POST</b> <i>/uno/player/pass-turn</i>| FD[<b>pass-turn-bff</b>]
    FD --> SA(dealer)
    SA -.->|Update the last card.| DLC[(last-card)]
    SA -.->|Query/Update the hand.| DOH[(opponent-hand)]
    SB(pile-card) -.->|Update the pile.| DPC[(pile-card)]
    SA -.->|If there is no card.| SB
    classDef first color:#fff,fill:green;
    classDef second color:#fff,fill:red;
    style U  color:#fff,fill:#000,stroke:#000;
    class FB,FC,FD first;
    class SA,SB,SC second;
```
</details>
