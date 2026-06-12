# Clean Architecture

> Tags: #tradeoffs #dx

Clean Architecture (Robert C. Martin) organizes code into concentric layers where **dependencies only point inward** — outer layers depend on inner layers, never the reverse.

---

## The Dependency Rule

> "Source code dependencies must point only inward, toward higher-level policies."

```
┌──────────────────────────────────┐
│         Frameworks & Drivers     │  ← Web, DB, UI, external APIs
│   ┌──────────────────────────┐   │
│   │     Interface Adapters   │   │  ← Controllers, Presenters, Gateways
│   │   ┌──────────────────┐   │   │
│   │   │  Application     │   │   │  ← Use Cases (business rules)
│   │   │  ┌────────────┐  │   │   │
│   │   │  │  Entities  │  │   │   │  ← Enterprise business rules
│   │   │  └────────────┘  │   │   │
│   │   └──────────────────┘   │   │
│   └──────────────────────────┘   │
└──────────────────────────────────┘
```

---

## The Four Layers

### 1. Entities (Enterprise Business Rules)
- Core business objects and rules that would exist regardless of the application
- Plain structs/classes with no framework dependencies
- Rarely change — only when fundamental business rules change

```go
// Example Entity (Go)
type Order struct {
    ID         string
    CustomerID string
    Items       []OrderItem
    Status     OrderStatus
    Total      Money
}

func (o *Order) AddItem(item OrderItem) error {
    if o.Status != OrderStatusDraft {
        return ErrOrderAlreadyConfirmed
    }
    o.Items = append(o.Items, item)
    o.Total = o.Total.Add(item.Price)
    return nil
}
```

### 2. Use Cases (Application Business Rules)
- Orchestrate the flow of data to/from entities to achieve a goal
- Define **input/output ports** (interfaces)
- Know nothing about HTTP, databases, or UI frameworks

```go
// Use Case interface
type PlaceOrderUseCase interface {
    Execute(ctx context.Context, input PlaceOrderInput) (PlaceOrderOutput, error)
}

// Input/Output DTOs
type PlaceOrderInput struct {
    CustomerID string
    Items      []OrderItemInput
}
type PlaceOrderOutput struct {
    OrderID string
    Total   float64
}
```

### 3. Interface Adapters
- Convert data from the format convenient for use cases to the format convenient for external agents
- Controllers, Presenters, Repository implementations, API clients

```go
// HTTP Controller (depends on use case interface, not implementation)
type OrderController struct {
    placeOrder PlaceOrderUseCase
}

func (c *OrderController) HandlePlaceOrder(w http.ResponseWriter, r *http.Request) {
    var req PlaceOrderRequest
    json.NewDecoder(r.Body).Decode(&req)
    output, err := c.placeOrder.Execute(r.Context(), toInput(req))
    // ...
}
```

### 4. Frameworks & Drivers
- The outermost layer: web frameworks, databases, message brokers, UI
- Where the actual implementation lives (e.g., SQL repository, HTTP router)
- Should be thin "glue code"

---

## Ports & Adapters (Hexagonal Architecture)

A related pattern that emphasizes the same idea:

- **Port**: an interface defined by the application (what it needs from the outside world)
- **Adapter**: concrete implementation of a port (what plugs into the port)

```
         [ HTTP Adapter ]  [ CLI Adapter ]
                  │               │
                  ▼               ▼
         ┌────────────────────────────┐
         │       Application Core     │
         │  (Use Cases + Entities)    │
         └────────────────────────────┘
                  │               │
                  ▼               ▼
         [ DB Adapter ]  [ Queue Adapter ]
```

---

## Dependency Injection

The mechanism that makes Clean Architecture work. Outer layers create inner implementations and inject them through interfaces.

**Composition Root** — a single place (usually `main`) where all dependencies are wired:

```go
// main.go — Composition Root
func main() {
    db := postgres.New(os.Getenv("DATABASE_URL"))
    orderRepo := repository.NewPostgresOrderRepository(db)
    placeOrder := usecase.NewPlaceOrderUseCase(orderRepo)
    controller := http.NewOrderController(placeOrder)
    // ...
}
```

---

## Testing Benefits

The dependency rule makes testing straightforward:

| Layer | Test Type | Dependencies |
|-------|-----------|-------------|
| Entities | Unit tests | None |
| Use Cases | Unit tests | Mocked ports (interfaces) |
| Interface Adapters | Integration tests | Real or test DB |
| Frameworks & Drivers | E2E / Contract tests | Full stack |

---

## Common Violations

| Violation | Problem |
|-----------|---------|
| Entity imports a DB package | Entity now depends on infrastructure |
| Use Case imports `http.Request` | Business logic tied to HTTP transport |
| Repository returns an ORM model | Domain leaks into storage details |
| Business logic in controllers | Untestable without HTTP stack |

---

## Comparison with Other Architectures

| | Layered (N-tier) | Hexagonal | Clean |
|-|:---:|:---:|:---:|
| Dependency direction | Top → Bottom | Core ↔ Ports | Inward only |
| Testability | Medium | High | High |
| Framework independence | Low | High | High |
| Learning curve | Low | Medium | Medium-High |

---

## When to Apply

✅ **Good fit:**
- Long-lived applications with complex business rules
- Multiple delivery mechanisms (HTTP, CLI, queues) over the same use cases
- Teams that need to test business logic in isolation

❌ **Overhead not worth it:**
- Simple CRUD services with no real business logic
- Prototypes or short-lived scripts

---

## References

- [Clean Architecture — Robert C. Martin (blog)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- Book: *Clean Architecture* — Robert C. Martin
- Book: *Domain-Driven Design* — Eric Evans
- [Alistair Cockburn — Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
