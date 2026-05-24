# Detection

How to identify anemic-domain-model candidates in the specified domain/context.

## Candidate requirements

Flag a candidate only when all are true:

1. The code is part of the domain model or directly mutates domain model state.
2. Business behavior exists: invariant, validation, calculation, policy, state transition, or lifecycle rule.
3. That behavior is outside the model that owns the relevant state, duplicated across callers, or hidden in application orchestration.

Do not flag:

- DTOs and request/response models
- persistence-only projections and read models
- generated classes
- simple CRUD records without meaningful business rules
- framework configuration classes
- mapper-only classes unless they contain domain rules

## Strong signals

### State mutation outside the owner

Service or handler code reaches into an entity and changes several fields directly.

```java
order.setStatus(OrderStatus.PAID);
order.setPaidAt(Instant.now());
order.setPaymentReference(reference);
```

Refactor direction:

```java
order.markPaid(reference, paidAt);
```

### Repeated caller checks

Multiple callers validate the same rule before mutating the same model.

```java
if (order.getStatus() != OrderStatus.DRAFT) {
    throw new IllegalStateException("Only draft orders can be submitted");
}
```

Refactor direction:

```java
order.submit(submittedAt);
```

### Exposed mutable collections

Callers add, remove, or replace children through getters.

```java
cart.getItems().add(new CartItem(productId, quantity));
```

Refactor direction:

```java
cart.addItem(productId, quantity);
```

### Constructor permits invalid state

Public setters or no-argument constructors allow incomplete objects beyond what the persistence tool requires.

```java
Order order = new Order();
order.setCustomer(customer);
order.setCurrency(currency);
```

Refactor direction:

```java
Order order = Order.openFor(customer, currency);
```

### Application service owns domain transitions

The application service decides the transition and applies all state changes itself.

```java
public void cancel(UUID orderId, String reason) {
    Order order = repository.get(orderId);
    if (order.isShipped()) throw new CannotCancelShippedOrder();
    order.setStatus(OrderStatus.CANCELLED);
    order.setCancellationReason(reason);
    repository.save(order);
}
```

Refactor direction:

```java
public void cancel(UUID orderId, String reason) {
    Order order = repository.get(orderId);
    order.cancel(reason);
    repository.save(order);
}
```

### Tests avoid the model

Tests can only verify domain behavior through mocked application services because entities expose no behavior.

Refactor direction: add tests at the domain model behavior method first, then slim down service tests.

## Severity guide

### Strong

- Multiple callers duplicate the same invariant or transition.
- Invalid states can be created through public constructors/setters.
- Exposed collections allow bypassing aggregate rules.
- Domain behavior is hard to discover and test without service orchestration.

### Worth exploring

- One application service owns several related rules that clearly belong to a single aggregate.
- A value object could remove primitive obsession and repeated validation.
- A factory could make valid creation easier and invalid creation harder.

### Speculative

- Behavior is minor, not duplicated, and current model is simple.
- The domain/context may be CRUD-only.
- A richer model would add ceremony without protecting meaningful invariants.

## Classification prompts

Use these prompts while exploring:

- Which object owns the state being changed?
- Which invariant can be violated if a caller forgets a check?
- Which business phrase does the code implement?
- Is this behavior repeated in more than one caller?
- Would a domain expert recognize the method name?
- Does this need a domain service because no entity/value object naturally owns it?
- Is this code actually a DTO, projection, or simple CRUD record?
