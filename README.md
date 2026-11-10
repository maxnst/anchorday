# Anchorday

A small Python library, with a command-line interface on top of it, for tracking **recurring obligations with a fixed anchor date**: a shared streaming plan, a gym membership, a shared utility bill, a rent split...

The core idea: every obligation has a **billing anchor** (the day the bill is actually due), and each participant's coverage is counted in billing cycles from that anchor. If someone pays late, their expiry date does **not** move: it stays tied to the group's cycle.

> **Status:** early planning. No code yet. This README describes the intended scope.

## Why this exists

Tracking shared recurring payments sounds simple until real calendars get involved: an anchor on the 31st, February, leap years, participants joining mid-cycle, payments made weeks after the cycle started. Most tools either ignore these cases or track the payment date instead of the coverage period. This project treats the date logic as the main product.

## Concepts

- **Group**: one recurring obligation. Free name, cost, currency, billing cycle (monthly or yearly) and anchor date.
- **Member**: a participant in a group. Has a first covered cycle and a list of payments.
- **Payment**: amount, the real date it was made, and the cycle it refers to. The two dates can differ.
- **Coverage**: up to which cycle a member has paid. Always derived from the group's anchor.
- **Status**: `covered`, `expiring`, or `overdue`, computed from coverage and today's date.

## Planned structure

```
src/anchorday/
  dates.py      calendar date arithmetic and month-end handling
  model.py      Group, Member, Payment
  coverage.py   coverage and status derivation
  ics.py        calendar file export (reminders)
  storage.py    local JSON persistence (kept outside the core logic)
  cli.py        command-line interface
tests/
pyproject.toml
```

## Design principles

- **Calendar dates, not timestamps.** Dates use the standard `datetime.date` (no time zone attached), so DST and time zones cannot shift an expiry.
- **Month-end handling is explicit.** An anchor on the 29th, 30th or 31st clamps to the last day of shorter months, and the rule is documented and tested.
- **The core stays pure.** `dates`, `model` and `coverage` have no clock, no file access and no network: "today" is always passed in.
- **Tests first for date logic.** Unit tests plus property-based tests for the edge cases.
- **Local only.** No accounts, no remote services, no analytics. Data stays in a local file.

## Roadmap

1. `dates` and `model`: anchor and cycle arithmetic, month-end clamping
2. `coverage`: coverage, status, renewal, retroactive anchor change
3. Full test suite (examples and property-based)
4. `storage` and `cli`: manage groups, members and payments from the terminal
5. `ics`: reminders through an exportable calendar file, no server needed
6. Optional: a small local web interface

## Future ideas

- One-off debts (a balance with partial payments) as a separate model, once the recurring core is complete and well tested

## Tech

Python 3.12+, pytest, Hypothesis (property-based testing), Typer (CLI), ruff (linting), mypy in strict mode.

## License

MIT
