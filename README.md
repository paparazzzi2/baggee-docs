# Baggee — product & system analysis

Analysis artifacts for **Baggee**, a P2P marketplace where buyers order goods from abroad and travelers bring them in their luggage. Payment is protected by **escrow**: money is held at the payment provider and released to the traveler only after the buyer confirms receipt.

This repository contains the *analysis layer* of the project (the "what and why"), not its source code:

| Document | What you will find |
|---|---|
| [Roles and permissions](docs/01-roles-and-permissions.md) | Actors, contextual buyer/seller roles, permission matrix, design principles |
| [User stories](docs/02-user-stories.md) | 6 epics, acceptance criteria, API mapping, backlog |
| [Deal process](docs/03-deal-process.md) | Lot and deal state machines, happy path and dispute sequence diagrams, guards, escrow rules |
| [Data model](docs/04-data-model.md) | ER diagram generated from the models (13 tables), design notes and known trade-offs |
| [API overview](docs/05-api-overview.md) | 96 operations in 12 groups, auth requirements |
| [`api/openapi.yaml`](api/openapi.yaml) | Machine-readable OpenAPI 3.1 contract (also as `openapi.json`) |

## How it was made

- The **OpenAPI contract and the ER diagram are generated from the FastAPI / SQLAlchemy code**, so they describe the real system, not an idea of it. UI page routes and mock payment stubs were removed; security-sensitive columns are left out of the ER diagram on purpose.
- The **process diagrams** (Mermaid, rendered by GitHub) were derived from the guards and side effects in the deal code. They are state machines and sequence diagrams, not formal BPMN.
- Where the analysis found a gap between the idea and the implementation (for example, route subscriptions are stored but not yet delivered as notifications), the documents say so and list it in the backlog.

## Skills demonstrated

Requirements and user stories · state-machine and process modelling · ER modelling · REST/OpenAPI contract design · role and permission analysis · payment/escrow domain (holds, idempotent webhooks, ledger).

## Related

- Public frontend (vanilla HTML/CSS/JS): [baggee-frontend](https://github.com/paparazzzi2/baggee-frontend)
- Live site: <https://baggee.ru>
- The backend is closed-source.

## Author

Mikael Sanoyan — Business Informatics student at NUST MISIS, aiming at systems analysis / product management. Telegram: <https://t.me/baggee_off>

## License

All rights reserved. Published for viewing and portfolio purposes only; copying and reuse require written permission. See [LICENSE](LICENSE).
