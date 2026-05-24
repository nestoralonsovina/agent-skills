# Refactoring

How to move from an anemic domain model toward a rich domain model safely.

## Default path

1. **Characterize current behavior.** Add focused tests around the existing rule, transition, or calculation before moving it.
2. **Name the behavior in domain language.** Prefer verbs the business uses: `submit`, `cancel`, `approve`, `settle`, `expire`, `reserve`.
3. **Move one behavior slice.** Start with one invariant or transition, not a whole aggregate rewrite.
4. **Protect the state.** Narrow setters, constructors, and collection access only as needed for the moved behavior.
5. **Slim the application service.** Leave loading, delegation, persistence, and side-effect coordination there.
6. **Run focused tests.** Verify after each slice.
7. **Remove duplication.** Delete old service-level checks only after domain tests cover the behavior.

## Behavior placement guide

### Entity

Use an entity when the rule controls identity, lifecycle, or state transitions.

```java
public class Order {
    private OrderStatus status;
    private Instant submittedAt;

    public void submit(Instant submittedAt) {
        if (status != OrderStatus.DRAFT) {
            throw new OrderCannotBeSubmitted(status);
        }

        this.status = OrderStatus.SUBMITTED;
        this.submittedAt = submittedAt;
    }
}
```

### Value object

Use a value object when validation or calculation belongs to a concept, not an entity lifecycle.

```java
public record Money(BigDecimal amount, Currency currency) {
    public Money {
        if (amount.signum() < 0) {
            throw new IllegalArgumentException("Money cannot be negative");
        }
        Objects.requireNonNull(currency);
    }

    public Money add(Money other) {
        if (!currency.equals(other.currency)) {
            throw new CurrencyMismatch(currency, other.currency);
        }
        return new Money(amount.add(other.amount), currency);
    }
}
```

### Aggregate

Use the aggregate when a rule protects consistency across child entities or value objects.

```java
public class Cart {
    private final List<CartItem> items = new ArrayList<>();

    public void addItem(ProductId productId, Quantity quantity) {
        if (items.size() >= 100) {
            throw new CartItemLimitReached();
        }

        items.add(new CartItem(productId, quantity));
    }

    public List<CartItem> items() {
        return List.copyOf(items);
    }
}
```

### Domain service

Use a domain service when the behavior has no natural owner on one entity/value object or spans aggregates.

```java
public class CreditPolicy {
    public CreditDecision evaluate(Customer customer, Order order, CreditExposure exposure) {
        if (exposure.exceedsLimitFor(customer)) {
            return CreditDecision.rejected("credit limit exceeded");
        }
        return CreditDecision.approved();
    }
}
```

The domain service is not a dumping ground. It must speak domain language and remain free of application workflow concerns.

### Domain event

Use a domain event when the domain needs to record that something happened and other parts of the system may react.

```java
public void submit(Instant submittedAt) {
    if (status != OrderStatus.DRAFT) {
        throw new OrderCannotBeSubmitted(status);
    }

    status = OrderStatus.SUBMITTED;
    this.submittedAt = submittedAt;
    registerEvent(new OrderSubmitted(id, submittedAt));
}
```

Application or infrastructure code handles email, queues, integrations, and retries.

## Java/Spring/JPA notes

- Keep a protected no-argument constructor only when JPA needs it.
- Prefer private fields and behavior methods over public setters.
- Return unmodifiable copies or read-only views for collections.
- Use static factories when creation has domain names or rules.
- Keep repository access out of entities. If a rule needs loaded data, load it in the application service and pass a domain object/value into the entity, or use a domain service.
- Keep `@Entity` and mapping decisions separate from the behavior-refactor decision.

## Test strategy

- Add domain tests for behavior methods and invariants.
- Keep application service tests thin: orchestration, transaction behavior, side effects, and repository interactions.
- Prefer testing through the domain method that callers use.
- Do not test private helpers introduced during refactoring.
- Remove tests that only lock in old anemic implementation details.

## Stop conditions

Stop and ask the user when:

- the behavior changes observable outcomes
- a public domain interface would break callers
- a domain term is unclear or conflicts with `CONTEXT.md`
- a rule could belong to more than one aggregate
- infrastructure constraints force a design trade-off
