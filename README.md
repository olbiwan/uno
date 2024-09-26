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

<p align="center">
  <img src="architecture-1.drawio.svg" alt="Alt text">
</p>

```mermaid
sequenceDiagram
    autonumber
    alt Logs in to the game.
      activate Player
        Player-->>First Level: POST /player/login
        First Level->>Player: token
      deactivate Player
    end
    Note left of Player: API available in REST format:
    alt Scenario: Invalid card played.
      activate Player
        Player-->>First Level: POST /player/refresh-token
        Player-->>First Level: POST /player/discard-card/:cardId
        First Level-->>Second Level: DELETE /uno/player-hand/:cardId
        Second Level->>First Level: Invalid card played (HTTP 409).
        First Level->>Player: Invalid card played (HTTP 409).
      deactivate Player
    end
    alt Draws a card from the deck and adds it to the player's hand.
      activate Player
        Player-->>First Level: POST /player/draw-card
        First Level-->>Second Level: POST /uno/draw-card
        Second Level->>First Level: cardId
        First Level-->>Second Level: POST /uno/player-hand
        First Level->>Player: cardId
      deactivate Player
    end
    alt Passes the turn to the next player.
      activate Player
        Player-->>First Level: POST /player/pass-turn
        First Level-->>Second Level: POST /uno/next-turn/:cardId
        Note right of First Level: Redis => "hash-next-turn".
        First Level->>Player: If there are more moves, return a list of the opponent's cards (HTTP 200).
        First Level->>Player: Indicates if the opponent has won (HTTP 410).
      deactivate Player
    end
```
