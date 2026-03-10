# 07 — Diagrams & Documentation

Generate Mermaid diagrams, API docs, onboarding materials, and keep them in sync with your code.

---

## What You'll Learn

- When and why to generate diagrams
- Mermaid syntax quick reference — just enough to read and edit Claude's output
- A catalog of diagram types with examples and when to use each one
- Generating documentation for modules, APIs, and onboarding
- Keeping generated docs in sync with code changes

**Prerequisites**: None — this guide is a reference you can use alongside any other guide.

---

## When to Generate Diagrams

Diagrams are most valuable when:

- **Onboarding** — you're learning a new codebase (or helping someone else learn)
- **Planning changes** — you need to see the blast radius visually
- **Code review** — a diagram explains a complex flow faster than prose
- **Documentation** — you're creating or updating project docs

Don't generate diagrams for their own sake. They should serve a specific purpose in the moment.

### The Prompt Pattern

```
Generate a Mermaid [diagram type] showing [what you want to see].
Include [specific details]. Keep it focused on [scope].
```

---

## Mermaid Syntax Quick Reference

You don't need to write Mermaid — Claude does that. But knowing enough to read and tweak its output is useful.

### Basic Shapes

```mermaid
flowchart LR
    A[Rectangle] --> B(Rounded)
    B --> C{Diamond}
    C -->|Yes| D[(Database)]
    C -->|No| E([Stadium])
```

```
flowchart LR
    A[Rectangle] --> B(Rounded)
    B --> C{Diamond}
    C -->|Yes| D[(Database)]
    C -->|No| E([Stadium])
```

### Arrow Types

```
A --> B       Solid arrow
A -.-> B      Dotted arrow
A ==> B       Thick arrow
A -->|label| B  Arrow with label
```

### Subgraphs

```
subgraph Title
    A --> B
end
```

That's enough to read and edit 90% of what Claude generates.

---

## Diagram Type Catalog

### Sequence Diagrams — Request Flows

Best for: tracing how a request moves through the system, showing interactions between components.

**When to use**: understanding API flows, debugging, PR descriptions for complex features.

```
Generate a Mermaid sequence diagram showing what happens
when a user submits a payment on the checkout page.
```

```mermaid
sequenceDiagram
    participant Browser
    participant API
    participant AuthMW as Auth Middleware
    participant PaySvc as Payment Service
    participant Stripe
    participant DB
    participant Queue

    Browser->>API: POST /api/checkout
    API->>AuthMW: validate session
    AuthMW-->>API: user context
    API->>PaySvc: processPayment(cart, user)
    PaySvc->>DB: lock inventory
    PaySvc->>Stripe: create charge
    Stripe-->>PaySvc: charge confirmed
    PaySvc->>DB: create order record
    PaySvc->>DB: release inventory lock
    PaySvc->>Queue: emit OrderCreated event
    PaySvc-->>API: order confirmation
    API-->>Browser: 201 Created + order details

    Note over Queue: Async: send confirmation email,<br/>update analytics, notify warehouse
```

### Flowcharts — Business Logic

Best for: decision trees, business rules, conditional logic, process flows.

**When to use**: understanding complex conditional code, documenting business rules, planning implementation.

```
Generate a Mermaid flowchart showing the order fulfillment
decision logic — how the system decides whether to ship,
backorder, or cancel.
```

```mermaid
flowchart TD
    A[New Order] --> B{All items<br/>in stock?}
    B -->|Yes| C{Total > $500?}
    B -->|No| D{Partial<br/>shipment OK?}
    C -->|Yes| E[Manual fraud review]
    C -->|No| F[Auto-approve]
    D -->|Yes| G[Ship available items<br/>Backorder remainder]
    D -->|No| H{Restock<br/>within 7 days?}
    H -->|Yes| I[Hold order<br/>Notify customer]
    H -->|No| J[Cancel + refund]
    E --> K{Approved?}
    K -->|Yes| F
    K -->|No| J
    F --> L[Create shipment]
    G --> L

    style J fill:#ffcdd2,stroke:#c62828
    style L fill:#c8e6c9,stroke:#2e7d32
```

### Class Diagrams — Object Relationships

Best for: understanding object models, inheritance hierarchies, interface contracts.

**When to use**: exploring a domain model, understanding design patterns in the code.

```
Generate a Mermaid class diagram showing the payment
processing domain model — classes, their relationships,
and key methods.
```

```mermaid
classDiagram
    class Order {
        +String id
        +OrderStatus status
        +LineItem[] items
        +Money total()
        +cancel()
        +fulfill()
    }

    class LineItem {
        +String productId
        +int quantity
        +Money unitPrice
        +Money subtotal()
    }

    class Payment {
        +String id
        +Money amount
        +PaymentMethod method
        +PaymentStatus status
        +process()
        +refund()
    }

    class PaymentMethod {
        <<interface>>
        +charge(Money amount)
        +refund(Money amount)
    }

    class CreditCard {
        +String last4
        +charge(Money amount)
        +refund(Money amount)
    }

    class BankTransfer {
        +String accountId
        +charge(Money amount)
        +refund(Money amount)
    }

    Order "1" --> "*" LineItem
    Order "1" --> "1" Payment
    Payment --> PaymentMethod
    PaymentMethod <|.. CreditCard
    PaymentMethod <|.. BankTransfer
```

### ER Diagrams — Data Models

Best for: database schema visualization, understanding table relationships.

**When to use**: before writing migrations, understanding the data layer, onboarding.

```
Generate a Mermaid ER diagram showing the core data model
for the user and permissions system.
```

```mermaid
erDiagram
    USERS ||--o{ USER_ROLES : has
    ROLES ||--o{ USER_ROLES : "assigned to"
    ROLES ||--o{ ROLE_PERMISSIONS : grants
    PERMISSIONS ||--o{ ROLE_PERMISSIONS : "granted by"

    USERS {
        uuid id PK
        string email UK
        string name
        boolean active
        timestamp last_login
    }

    ROLES {
        uuid id PK
        string name UK
        string description
    }

    USER_ROLES {
        uuid user_id FK
        uuid role_id FK
        timestamp granted_at
    }

    PERMISSIONS {
        uuid id PK
        string resource
        string action
    }

    ROLE_PERMISSIONS {
        uuid role_id FK
        uuid permission_id FK
    }
```

### Architecture Diagrams — System Overview

Best for: showing services, data stores, and how they connect at a high level.

**When to use**: onboarding, system design discussions, understanding deployment.

See [Guide 04](04-architecture-and-dependencies.md) for detailed architecture diagram examples.

### State Diagrams — Workflows and State Machines

Best for: entities with lifecycle states, workflow processes, protocol states.

**When to use**: understanding order states, ticket lifecycles, document workflows.

```
Generate a Mermaid state diagram showing the lifecycle
of an order from creation to completion.
```

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Submitted: customer submits
    Submitted --> Processing: payment confirmed
    Submitted --> Cancelled: payment failed

    Processing --> Shipped: items dispatched
    Processing --> PartiallyShipped: some items dispatched

    PartiallyShipped --> Shipped: remaining items dispatched
    PartiallyShipped --> PartialRefund: remaining items cancelled

    Shipped --> Delivered: delivery confirmed
    Shipped --> ReturnRequested: customer requests return

    Delivered --> ReturnRequested: customer requests return
    Delivered --> Completed: return window expires

    ReturnRequested --> Returned: return received
    Returned --> Refunded: refund processed

    Cancelled --> [*]
    Completed --> [*]
    Refunded --> [*]
```

### Gitgraph Diagrams — Branch Strategies

Best for: explaining branching strategies, visualizing release flows.

**When to use**: onboarding docs, release process documentation.

```
Generate a Mermaid gitgraph diagram showing this project's
branching strategy based on the git history.
```

```mermaid
gitGraph
    commit id: "initial"
    branch develop
    commit id: "feature work"
    branch feature/auth
    commit id: "add login"
    commit id: "add signup"
    checkout develop
    merge feature/auth id: "merge auth"
    branch feature/payments
    commit id: "add stripe"
    checkout develop
    merge feature/payments id: "merge payments"
    checkout main
    merge develop id: "v1.2.0" tag: "v1.2.0"
```

---

## Generating Documentation

### Module Documentation

```
Generate documentation for the [module/feature] including:
- Purpose and responsibilities
- Key interfaces and data structures
- Sequence diagram of the main flow (in Mermaid)
- Known limitations or gotchas
- Examples of how to use it
```

### API Documentation

```
Generate API documentation for the endpoints in [file/directory]:
- Method, path, description
- Request body schema with examples
- Response format with examples
- Error responses
- Authentication requirements
```

### Onboarding Documentation

Turn your exploration sessions into onboarding materials:

```
Based on everything we've explored in this session, create
an onboarding document for new developers. Include:
- Project overview and architecture
- How to set up the development environment
- Key concepts a new developer needs to understand
- Common tasks and how to do them
- Links to important files and directories
- Gotchas and tribal knowledge
```

---

## Keeping Docs in Sync

Generated documentation drifts from reality as code changes. Here are strategies to keep them aligned.

### Periodic Auditing

```
Compare what's documented in [README/docs/wiki] against what
the code actually does. Are there discrepancies? What's
outdated? What's missing?
```

### Post-Change Updates

After making significant code changes:

```
We just changed [what you changed]. Check if any existing
documentation needs to be updated to reflect these changes.
Check README.md, any docs/ files, and CLAUDE.md.
```

### Documentation as Code

The most maintainable diagrams are the ones that live close to the code they describe:

- Keep Mermaid diagrams in markdown files next to the code
- Reference them from the main README
- Update them as part of the PR when the code changes

---

## Key Takeaways

1. Generate diagrams to serve a purpose (onboarding, planning, review) — not for their own sake
2. Sequence diagrams and flowcharts are the most universally useful types
3. You don't need to memorize Mermaid syntax — just know enough to read and tweak Claude's output
4. Turn exploration sessions into onboarding docs — the work is already done
5. Keep docs close to code, and update them when the code changes

---

**Next**: [08 — Ongoing Practices](08-ongoing-practices.md) — Build sustainable habits for long-term productivity.
