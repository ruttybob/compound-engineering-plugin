# CONCEPTS.md vocabulary rules

`CONCEPTS.md` defines the words that mean something specific in this codebase — substrate that `docs/solutions/` and AGENTS.md can cite without redefinition. Lives at the repo root, created lazily the first time a learning surfaces a qualifying term.

## What earns a slot

A term qualifies when its meaning here is precise enough that a new engineer would need it defined to follow conversations, tickets, or code. General programming vocabulary — caches, queues, jobs, sessions — does not belong, even when used heavily.

## What never appears

Implementation specifics — file paths, class names, table names, function signatures. Status fields, dates, owners on the entries. Examples drawn from current code. Anything git, the codebase, or the learnings store would tell you.

## Per entry

Lead with identity, not behavior — what kind of thing it is, what makes it distinct, what it stands in relation to. Length tracks complexity: most entries are one sentence; a term with non-obvious rules earns a second paragraph for them. Entities typically need more depth than value types; status concepts may need transition notes. When the team uses several words for the same concept, choose one and retire the rest.

## Organization

Cluster concepts by domain relationship — entities with their states, processes with their stages — so a reader sees structure without effort. A flat list works when the file is small. Reshape as the file grows.

## Resolved ambiguities (optional, tail of file)

When two terms were used interchangeably and the team settled on a distinction, record the resolution as a one-line note: *"'account' had been used for both Customer and User — these are distinct."*

## One illustrative entry — the shape, not a template

```
## Booking

### Reservation
A future commitment to seat a Party at a specified date and time. A Reservation owns its Party but does not own a Table — Tables are acquired only when the Party arrives, through a Seating. Lifecycle: Booked, Seated, Completed, No-Show.

Cancellation before a Seating is non-destructive. Cancellation after a Seating is recorded as a No-Show.

### Party
The guests committed to a Reservation. Each Reservation has exactly one Party. Party size is the count promised at booking, not the count who arrive.

### Table
A physical seating unit with fixed capacity. Tables are shared resources — they do not belong to Reservations and are allocated only on the day-of through Seatings.

### Seating
The act of placing a Party at a Table once the Party arrives. A Reservation has at most one Seating; a Table accumulates many Seatings across its lifetime.
```
