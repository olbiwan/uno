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
    participant web as First Level
    participant blog as Blog Service
    participant account as Account Service
    participant mail as Mail Service
    participant db as Storage

    Note over web,db: The user must be logged in to submit blog posts
    web->>+account: Logs in using credentials
    account->>db: Query stored accounts
    db->>account: Respond with query result

    alt Credentials not found
        account->>web: Invalid credentials
    else Credentials found
        account->>-web: Successfully logged in

        Note over web,db: When the user is authenticated, they can now submit new posts
        web->>+blog: Submit new post
        blog->>db: Store post data

        par Notifications
            blog--)mail: Send mail to blog subscribers
            blog--)db: Store in-site notifications
        and Response
            blog-->>-web: Successfully posted
        end
    end
```
