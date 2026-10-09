# Roles and permissions

Baggee has **three account roles** stored on the user (`user`, `moderator`, `admin`) and one anonymous visitor. A user is not "buyer" or "seller" by account type: the role is **contextual**, it depends on the deal. The traveler who owns a lot acts as the *seller* in that lot's deals; whoever pays acts as the *buyer*.

## Actors

| Actor | Who | Notes |
|---|---|---|
| **Visitor** | Not signed in | Browses lots, reads reviews and public pages, registers, logs in |
| **User** | Signed-in account (`user`) | Needs a **verified e-mail** to publish a lot. Plays *seller* or *buyer* depending on the deal |
| **Moderator** | `moderator` | Staff: lot moderation, payment checks, support and disputes |
| **Admin** | `admin` | Everything a moderator can do plus role management, review removal, balance adjustments and wallet audit |
| **Payment provider** | External system | Holds, releases and refunds money; sends webhooks |

## Permission matrix

Legend: ✅ allowed, — not allowed, *own* = only for own objects, *party* = only when the user is a participant of the deal.

| Capability | Visitor | User | Moderator | Admin |
|---|:--:|:--:|:--:|:--:|
| Browse lots, view a lot, view user reviews | ✅ | ✅ | ✅ | ✅ |
| Register, log in, confirm e-mail, reset password | ✅ | ✅ | ✅ | ✅ |
| Publish a lot (goes to moderation) | — | ✅ (verified e-mail) | ✅ | ✅ |
| Edit / delete a lot, attach trip tickets | — | *own* | — | — |
| Approve or reject a pending lot | — | — | ✅ | ✅ |
| Create a deal in a lot | — | *own lot* | — | — |
| Pay a deal (escrow checkout) | — | *buyer* | — | — |
| Confirm or reject a manual payment proof | — | — | ✅ | ✅ |
| Upload item photos | — | *seller* | — | — |
| Confirm photos, confirm receipt (release money) | — | *buyer* | — | — |
| Mark "I have arrived" | — | *seller* | — | — |
| Cancel a deal | — | *party* | ✅ | ✅ |
| Open a dispute / call support | — | *party* | ✅ | ✅ |
| Read the buyer-seller chat history of a deal | — | — | ✅ | ✅ |
| Write in the deal chat as support, close a ticket | — | — | ✅ | ✅ |
| Resolve a dispute (release or refund) | — | — | ✅ | ✅ |
| Direct messages, notifications, route alerts | — | ✅ | ✅ | ✅ |
| Leave a review about a user (one per reviewer, user and lot) | — | ✅ | ✅ | ✅ |
| Save payout details, request a withdrawal, view own ledger | — | ✅ | ✅ | ✅ |
| View the list of withdrawal requests | — | — | ✅ | ✅ |
| Approve or reject a withdrawal | — | — | — | ✅ |
| Manual balance adjustment, wallet audit | — | — | — | ✅ |
| Change user roles, delete reviews | — | — | — | ✅ |

## Design principles visible in the permissions

1. **Separation of duties.** The seller proves delivery (photos), the buyer releases the money, the moderator only intervenes on a dispute or a manual payment. No single role can both receive and release the funds.
2. **Server-side checks.** Role and ownership checks live in the backend; the public frontend only hides buttons.
3. **Least privilege for staff.** Withdrawal destination tokens are never returned to staff, even to admins; money adjustments are admin-only and always logged with the author.
4. **Auditability.** Every balance change is a ledger entry; payments keep timestamps for hold, release and refund.
