# F1. Distributed Transaction Fail Flow & Saga Pattern

**Context:** In a fintech payment system, a single transfer may touch multiple services:

```
Client → Payment Service (A) → Ledger Service (B) → Notification Service (C) → Audit Service (D)
```

If step C fails, how do you ensure the system stays consistent?

---

## The core problem

In a monolith, `@Transactional` rolls everything back atomically. In microservices, each service has its own database — **there is no global transaction manager**. You cannot do a 2-phase commit across service boundaries at scale.

---

## What is the Saga Pattern?

A **Saga** is a sequence of local transactions, each publishing an event or message that triggers the next step. If any step fails, **compensating transactions** undo the previous steps.

```
Step 1: debit sender account        → success → emit "SenderDebited"
Step 2: credit receiver account     → success → emit "ReceiverCredited"
Step 3: send notification           → FAIL    → emit "NotificationFailed"

Compensate step 2: reverse credit   → emit "ReceiverCreditReversed"
Compensate step 1: reverse debit    → emit "SenderDebitReversed"
```

Each step is **atomic locally**. Consistency is achieved **eventually**, not instantly.

---

## Two types of Saga

### 1. Choreography (event-driven)

Services communicate by publishing and subscribing to events. No central controller.

```
PaymentService  ──[SenderDebited]──►  LedgerService
LedgerService   ──[ReceiverCredited]──►  NotifService
NotifService    ──[NotifFailed]──►  LedgerService (compensate)
LedgerService   ──[ReceiverReversed]──►  PaymentService (compensate)
```

**Pros:**
- Loose coupling — services don't know about each other
- Easy to add a new step without changing existing services

**Cons:**
- Hard to trace the overall flow (events scattered across services)
- Risk of cyclic event loops if not careful
- No single place to see the transaction state

**Best for:** simple flows with few steps, or when services are truly independent teams.

---

### 2. Orchestration (central coordinator)

A dedicated **Saga Orchestrator** (a service or state machine) tells each participant what to do next and handles failures.

```java
// Pseudo-code orchestrator
@Service
public class TransferSagaOrchestrator {

    public void execute(TransferCommand cmd) {
        try {
            paymentService.debitSender(cmd);        // step 1
            ledgerService.creditReceiver(cmd);       // step 2
            notifService.sendNotification(cmd);      // step 3
            auditService.logTransfer(cmd);           // step 4
        } catch (NotifException e) {
            // compensate backwards
            ledgerService.reverseCredit(cmd);
            paymentService.reverseDebit(cmd);
            throw new TransferFailedException(cmd.getId());
        }
    }
}
```

With a state machine (e.g. using Spring State Machine or a workflow engine like Temporal):

```
States: INITIATED → DEBITED → CREDITED → NOTIFIED → COMPLETED
                                        ↓ (on fail)
                             COMPENSATING → REVERSED → FAILED
```

**Pros:**
- Centralized visibility — easy to see where a transaction is
- Easy to add retry logic and timeouts
- Clear compensation flow

**Cons:**
- Orchestrator becomes a new point of failure
- Tighter coupling — orchestrator knows about all services

**Best for:** complex multi-step flows (payments, loans, onboarding) where visibility and control matter.

---

## Fintech example: money transfer A → B → C → D

```
A: PaymentService   — validate & debit source account
B: LedgerService    — credit destination account
C: FXService        — apply currency conversion (if cross-currency)
D: AuditService     — write compliance log
```

Failure scenarios:
| Fails at | Compensation needed |
|---|---|
| B | reverse A (refund source) |
| C | reverse B (reverse credit), reverse A (refund source) |
| D | reverse C, reverse B, reverse A |

**Key insight:** AuditService (D) should be idempotent and retried, not compensated. You never want to "un-audit" a financial transaction — instead, write a corrective audit entry.

---

## Making it reliable: Outbox Pattern + Saga

The biggest risk in Saga is: what if the service crashes *after* the local transaction commits but *before* the event is published?

Solution: combine Saga with the **Outbox Pattern**:

```java
@Transactional
public void debitSender(TransferCommand cmd) {
    accountRepo.debit(cmd.getSenderId(), cmd.getAmount());  // local DB write
    outboxRepo.save(new OutboxEvent("SenderDebited", cmd)); // same transaction
    // Relay picks up outbox → publishes to Kafka → at-least-once delivery
}
```

This guarantees: **either both the debit and the event are written, or neither is.**

---

## Idempotency is mandatory

Because Saga steps can be retried (network timeout, crash), every step must be idempotent:

```java
// Check if this transfer was already processed
if (ledgerRepo.existsByTransferId(cmd.getTransferId())) {
    return; // already applied, skip
}
ledgerRepo.creditReceiver(cmd);
```

Use the `transferId` (UUID) as the idempotency key everywhere.

---

## Choreography vs Orchestration — when to choose

| | Choreography | Orchestration |
|---|---|---|
| Flow visibility | Hard | Easy |
| Coupling | Loose | Tighter |
| Complexity scales with | # of services | # of steps |
| Best fit | Simple, independent steps | Complex financial flows |
| Real fintech usage | Notification, audit side effects | Core payment/transfer flows |

---

## Interview one-liner
> "When a distributed transaction spans multiple services (debit → credit → notify → audit), a database-level rollback isn't possible. The Saga pattern solves this by breaking the flow into local transactions with compensating transactions for rollback. Choreography uses events between services (loose coupling, hard to trace); Orchestration uses a central coordinator (visible, controllable). In fintech, orchestration is usually preferred for core money movement, and we combine it with the Outbox pattern to guarantee event delivery and idempotency keys to make each step safe to retry."
