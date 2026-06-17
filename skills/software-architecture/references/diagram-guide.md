# Diagram Guide

Diagrams go **inline in the document as Mermaid code blocks**, so the document
stays a single portable artifact that renders on GitHub, GitLab, and most
Markdown viewers. Use the **C4 model** as the mental framework for *what* to
draw, and (by default) plain `flowchart`/`graph` syntax for *how* to draw it.

## The C4 model in one minute

C4 is a set of nested zoom levels. You rarely need all four:

1. **System Context** — the system as one box, surrounded by its users and the
   external systems it talks to. Answers "what is this and who/what does it
   interact with." *Almost always worth drawing.*
2. **Container** — zoom into the system to show its deployable/runnable units
   (web app, API, database, queue, etc.) and how they communicate. "Container"
   here means a runtime unit, not a Docker container specifically. *The single
   most useful architecture diagram; almost always draw this.*
3. **Component** — zoom into one container to show its internal parts. Draw only
   for containers complex enough to need it.
4. **Code** — class-level. Almost never worth maintaining by hand; skip.

Draw Level 1 and Level 2 by default. Add Level 3 only where a container is
genuinely intricate.

## Rendering reliability — read this first

Mermaid has dedicated `C4Context`/`C4Container` syntax, but its rendering is
still experimental and is **unreliable on GitHub and several common viewers**.
For a portable document, **default to `flowchart`/`graph` with subgraphs**,
styled to convey the C4 levels. It renders everywhere. Use the native C4 syntax
only if the user specifically wants it and you know their viewer supports it.

Two more reliability rules that prevent most broken diagrams:
- **Always quote node text** that contains spaces, punctuation, or special
  characters: `api["API Service"]`, `db[("PostgreSQL")]`.
- Keep edge labels short and quote them: `web -->|"REST/JSON"| api`.

---

## System Context (C4 Level 1) — reliable pattern

```mermaid
graph TB
    user["Customer<br/>(person)"]
    admin["Staff<br/>(person)"]
    system["<b>Order System</b><br/>Lets customers browse and<br/>place orders"]
    payment["Payment Provider<br/>(external)"]
    email["Email Service<br/>(external)"]

    user -->|"Browses, orders"| system
    admin -->|"Manages catalog"| system
    system -->|"Charges cards"| payment
    system -->|"Sends receipts"| email

    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef internal fill:#1168bd,stroke:#0b4884,color:#fff
    classDef external fill:#999,stroke:#666,color:#fff
    class user,admin person
    class system internal
    class payment,email external
```

## Container (C4 Level 2) — reliable pattern

The workhorse. Show the runnable units inside the system boundary and the
external dependencies they touch.

```mermaid
graph TB
    user["Customer"]

    subgraph system["Order System"]
        web["Web App<br/>[Next.js]"]
        api["API Service<br/>[Node.js]"]
        worker["Worker<br/>[Node.js]<br/>processes async jobs"]
        db[("PostgreSQL<br/>orders, catalog")]
        queue["Message Queue<br/>[SQS]"]
    end

    payment["Payment Provider<br/>(external)"]
    email["Email Service<br/>(external)"]

    user -->|"HTTPS"| web
    web -->|"REST/JSON"| api
    api -->|"reads/writes"| db
    api -->|"enqueues jobs"| queue
    queue -->|"delivers"| worker
    worker -->|"reads/writes"| db
    api -->|"charges"| payment
    worker -->|"sends mail"| email

    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef container fill:#438dd5,stroke:#2e6295,color:#fff
    classDef store fill:#438dd5,stroke:#2e6295,color:#fff
    classDef external fill:#999,stroke:#666,color:#fff
    class user person
    class web,api,worker,queue container
    class db store
    class payment,email external
```

## Runtime / sequence diagram

For the Runtime View. Sequence diagrams are well-supported and render reliably
everywhere — no special care needed beyond keeping them focused on one scenario.

```mermaid
sequenceDiagram
    actor User
    participant Web as Web App
    participant API as API Service
    participant DB as PostgreSQL
    participant Q as Queue
    participant Pay as Payment Provider

    User->>Web: Submit order
    Web->>API: POST /orders
    API->>DB: Insert order (status=pending)
    API->>Pay: Charge card
    Pay-->>API: Payment confirmed
    API->>DB: Update order (status=paid)
    API->>Q: Enqueue "send receipt"
    API-->>Web: 201 Created
    Web-->>User: Order confirmed
```

## Deployment diagram

For the Deployment View, when topology is non-trivial. Group infrastructure with
subgraphs representing regions, zones, or networks.

```mermaid
graph TB
    subgraph cloud["AWS (us-east-1)"]
        subgraph vpc["VPC"]
            subgraph public["Public subnet"]
                lb["Load Balancer"]
            end
            subgraph private["Private subnet"]
                app["App containers<br/>(ECS Fargate, 2+ tasks)"]
                rds[("RDS PostgreSQL<br/>Multi-AZ")]
            end
        end
        cdn["CloudFront CDN"]
    end
    users["Users"]

    users -->|"HTTPS"| cdn
    cdn --> lb
    lb --> app
    app --> rds
```

---

## State diagram (optional)

When an entity has a meaningful lifecycle worth documenting (an order, a
subscription, a deployment), a state diagram earns its place:

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Paid: payment confirmed
    Pending --> Cancelled: timeout / user cancels
    Paid --> Shipped: fulfillment
    Shipped --> Delivered
    Paid --> Refunded: refund issued
    Delivered --> [*]
    Cancelled --> [*]
    Refunded --> [*]
```

---

## Diagram checklist

- Did you draw at least a Context and a Container diagram? (Default minimum.)
- Is every external system shown as external, and every internal unit inside the
  system boundary?
- Does every edge have a short, meaningful label (what flows, and how)?
- Did you quote all node text with spaces/punctuation?
- Will it render on the user's target viewer? (Default to `graph`/`flowchart`
  for portability; only use native `C4*` syntax when you know it's supported.)
- Is the diagram telling the same story as the prose around it?
