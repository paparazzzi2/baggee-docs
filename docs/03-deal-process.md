# Deal process (escrow)

How a deal moves from offer to payout. Everything below is derived from the backend code (statuses, guards and side effects), not from a wish-list.

**Actors:** *Seller* = the traveler who owns the lot and brings the item, *Buyer* = the person who orders it, *Moderator* (support), *Payment provider* (escrow hold at the acquirer), *Platform* (the Baggee backend).

## 1. Lot lifecycle

A lot is a traveler's trip offer. It is hidden from the catalog until a moderator approves it.

```mermaid
stateDiagram-v2
    [*] --> pending: Traveler publishes lot (e-mail must be verified)
    pending --> active: Moderator approves
    pending --> rejected: Moderator rejects
    active --> in_deal: Free weight reaches 0 after a paid deal
    in_deal --> active: Weight is returned (cancel / refund)
    rejected --> [*]
```

## 2. Deal states

The seller creates a deal **inside their own lot** (item, price, weight; optionally for a specific buyer). The platform adds its commission to the item price and checks that the requested weight fits the lot's free weight. The weight is **deducted from the lot only when the deal becomes paid**, so unpaid offers never block baggage space; if the free weight drops to zero the lot switches to `in_deal`.

```mermaid
stateDiagram-v2
    [*] --> created: Seller creates deal
    created --> payment_uploaded: Buyer uploads proof of payment<br/>(manual flow)
    payment_uploaded --> paid: Moderator confirms payment
    payment_uploaded --> created: Moderator rejects payment
    created --> paid: Escrow hold succeeded at provider<br/>(webhook, checked by pull)<br/>weight deducted from the lot
    paid --> photos_uploaded: Seller uploads item photos (up to 5)
    photos_uploaded --> photos_confirmed: Buyer confirms the photos
    photos_confirmed --> completed: Buyer confirms receipt<br/>=> money released to seller
    paid --> disputed: Buyer or seller opens dispute<br/>(only after payment)
    photos_uploaded --> disputed
    photos_confirmed --> disputed
    disputed --> completed: Moderator rules for seller<br/>=> release
    disputed --> cancelled: Moderator rules for buyer<br/>=> refund, weight returned to the lot
    created --> cancelled: Participant or moderator cancels
    paid --> cancelled
    photos_uploaded --> cancelled
    photos_confirmed --> cancelled
    completed --> [*]
    cancelled --> [*]
```

Side flags that do **not** change the status: `seller_arrived` (one-time "I have landed" signal to the buyer) and `support_called` / `support_closed` (a support ticket can be opened and closed on a deal without changing its state).

### Guards worth noting (from the code)

| Rule | Where it matters |
|---|---|
| Only the lot owner can create deals; the lot must be `active`; requested weight must not exceed the lot's free weight | Only the traveler decides what they bring; offers cannot exceed the capacity |
| Weight is deducted on payment (idempotent flag `weight_deducted`) and restored on cancel or refund only if it was deducted | No double deduction, no double return |
| A dispute can be opened only after payment, when a buyer exists and money is in escrow | Support would have nobody to refund otherwise |
| `complete` is allowed only for the buyer and only from `photos_confirmed` | The buyer, not the seller, decides when money is released |
| Cancel is allowed to participants and moderators unless the deal is closed; it refunds a held payment and returns the weight to the lot | Keeps lot capacity and money consistent |
| Provider webhooks are **idempotent** (event ids are stored) and the platform **does not trust the webhook body**: it asks the provider for the payment status | Protects against duplicated or forged events |
| A failed or cancelled bank payment marks the payment `failed` and notifies the buyer; the deal stays `created` so the buyer can retry | Graceful payment failure |
| A demo "pay now" shortcut exists and is **disabled when real payments are on** | Safe separation of demo and production |

## 3. Happy path (sequence)

```mermaid
sequenceDiagram
    autonumber
    actor S as Seller (traveler)
    actor B as Buyer
    participant P as Platform (backend)
    participant PR as Payment provider
    actor M as Moderator

    S->>P: Publish lot (route, weight, commission)
    M->>P: Approve lot
    P-->>S: Lot is active
    S->>P: Create deal (item, price, weight)
    P->>P: final price = price x (1 + commission%)
    P-->>B: Notification: deal offered
    B->>P: Start checkout
    P->>PR: Create payment with hold (escrow)
    PR-->>B: Confirmation page (bank)
    B->>PR: Pay
    PR->>P: Webhook "payment held"
    P->>PR: Pull payment status (do not trust the body)
    P->>P: Deal -> paid, deduct weight from the lot, notify both sides
    S->>P: Upload photos of the item
    B->>P: Confirm photos
    S->>P: "I have arrived" (flag)
    S-->>B: Hand over the item in person
    B->>P: Confirm receipt
    P->>PR: Release payment (minus platform commission)
    P->>P: Deal -> completed, credit / payout to seller
    P-->>B: Invite to leave a review
```

## 4. Dispute branch (sequence)

```mermaid
sequenceDiagram
    autonumber
    actor X as Buyer or Seller
    participant P as Platform
    actor M as Moderator
    participant PR as Payment provider

    X->>P: Open dispute with a reason (only after payment)
    P->>P: Deal -> disputed, notify the other side and support
    M->>P: Read the deal thread and the peer chat history
    M->>P: Write to the deal chat as support
    alt Moderator rules for the seller
        M->>P: resolve(winner = seller)
        P->>PR: Release payment
        P->>P: Deal -> completed
    else Moderator rules for the buyer
        M->>P: resolve(winner = buyer)
        P->>PR: Refund payment
        P->>P: Deal -> cancelled, weight returned to the lot
    end
    P->>P: Decision text is appended to the dispute log, support ticket closed
```

## 5. Money after completion

With the default payout mode the seller is paid when the deal closes (the platform does not hold other people's money longer than the escrow). An alternative mode credits an internal balance and the seller withdraws by request; withdrawals go through limits, a cooldown and optional manual review, and every balance change is written to the ledger. The legal consequences of holding balances (licensing) are the reason the first mode is the recommended default.
