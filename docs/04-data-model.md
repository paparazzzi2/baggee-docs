# Data model (ER)

Entity-relationship diagram of the Baggee database, **generated from the SQLAlchemy models** (13 tables, 15 declared foreign keys).
Solid lines are real foreign keys; dotted lines are *logical* links that the code maintains but the database does not enforce (for example `deals.buyer_username`, `messages.deal_id`, `ledger_entries.deal_id`).

> Security- and privacy-sensitive columns (password hash, e-mail hash/ciphertext, payout tokens, IP addresses, raw provider payloads, idempotence keys) are **intentionally omitted** from the diagram (22 columns).

```mermaid
erDiagram
    deals {
        int id PK
        int lot_id FK
        string seller_username FK
        string buyer_username
        string target_buyer_username
        text item_description
        string item_photo
        float item_price
        float commission_pct
        float final_price
        float weight_kg
        string category
        string status
        string payment_proof
        bool weight_deducted
        int payment_id FK
        string seller_photos
        bool seller_arrived
        bool support_called
        bool support_ever
        bool support_closed
        string support_caller
        text dispute_reason
        datetime created_at
        datetime paid_at
        datetime completed_at
    }
    ledger_entries {
        int id PK
        int user_id FK
        string kind
        int amount_minor
        int balance_after_minor
        int deal_id
        int payment_id
        int withdrawal_id
        string ref_key
        string comment
        string actor_username
        datetime created_at
    }
    lots {
        int id PK
        string owner_username FK
        string country_from
        string country_to
        string city_from
        string city_to
        date arrival_date
        float weight_kg
        float commission_pct
        int slots_left
        string brands
        string category
        text description
        string tickets
        string status
        datetime created_at
    }
    messages {
        int id PK
        string sender_username FK
        string receiver_username FK
        int lot_id
        int deal_id
        bool is_support
        text text
        datetime created_at
        datetime read_at
    }
    notifications {
        int id PK
        int user_id FK
        text message
        bool is_read
        datetime created_at
    }
    payments {
        int id PK
        int deal_id FK
        string provider
        string status
        int amount_minor
        int commission_minor
        int payout_minor
        string currency
        datetime created_at
        datetime held_at
        datetime released_at
        datetime refunded_at
    }
    reviews {
        int id PK
        int reviewer_id FK
        int reviewed_id FK
        int lot_id FK
        int rating
        text text
        datetime created_at
    }
    route_alerts {
        int id PK
        int user_id FK
        string country_from
        string city_from
        string country_to
        string city_to
        date date_from
        date date_to
        float min_weight
        datetime created_at
    }
    site_visits {
        int id PK
        string day
        int hits
        datetime first_ts
        datetime last_ts
    }
    user_consents {
        int id PK
        int user_id FK
        string doc_type
        string doc_version
        datetime accepted_at
    }
    users {
        int id PK
        string username
        int email_verified
        datetime registration_date
        datetime last_seen
        int balance_minor
        string role
        string nationality
        string payout_method
        string payout_card_type
        datetime payout_bound_at
    }
    webhook_events {
        int id PK
        string event_id
        string provider
        int payment_id
        datetime created_at
    }
    withdrawals {
        int id PK
        int user_id FK
        string username
        int amount_minor
        int fee_minor
        int payout_minor
        string currency
        string status
        string method
        string dest_masked
        string provider
        string reject_reason
        string reviewed_by
        datetime created_at
        datetime processed_at
        datetime paid_at
    }
    users ||--o{ lots : "owner_username (publishes)"
    lots ||--o{ deals : "lot_id"
    users ||--o{ deals : "seller_username"
    users |o..o{ deals : "buyer_username (logical)"
    deals |o--o| payments : "payment_id / deal_id (escrow)"
    payments ||..o{ webhook_events : "payment_id (logical)"
    users ||--o{ messages : "sender_username"
    users ||--o{ messages : "receiver_username"
    deals |o..o{ messages : "deal_id (logical, deal chat)"
    lots |o..o{ messages : "lot_id (logical)"
    users ||--o{ reviews : "reviewer_id"
    users ||--o{ reviews : "reviewed_id"
    lots ||--o{ reviews : "lot_id"
    users ||--o{ notifications : "user_id"
    users ||--o{ route_alerts : "user_id"
    users ||--o{ user_consents : "user_id"
    users ||--o{ withdrawals : "user_id"
    users ||--o{ ledger_entries : "user_id"
    deals |o..o{ ledger_entries : "deal_id (logical)"
    withdrawals |o..o{ ledger_entries : "withdrawal_id (logical)"
```

## Tables at a glance

| Table | Purpose |
|---|---|
| `users` | Accounts, roles (`user` / `moderator` / `admin`), balance in minor units (kopecks), payout method |
| `lots` | A traveler's trip offer: route, arrival date, free weight, platform commission %, moderation status (`pending` → `active` / `rejected`) |
| `deals` | One item deal inside a lot: item, price, final price with commission, weight reserved, status, support/dispute flags |
| `payments` | Escrow payment of a deal at the payment provider: amounts in minor units, commission, payout, lifecycle timestamps (`held_at`, `released_at`, `refunded_at`) |
| `webhook_events` | Processed provider events, used for idempotency (the same event is never applied twice) |
| `messages` | Direct chats and the deal chat, including support (moderator) messages |
| `reviews` | Ratings and text between users, tied to a lot |
| `notifications` | In-app notifications |
| `route_alerts` | A user's subscription to routes / dates / minimum weight |
| `user_consents` | Consent log: which legal document version a user accepted and when (relevant for 152-FZ) |
| `ledger_entries` | Journal of balance changes (credit, adjustment, withdrawal) with the balance after each operation and an idempotency key |
| `withdrawals` | Withdrawal requests: `pending` → `processing` → `paid` (or cancelled / rejected), optionally with manual staff review |
| `site_visits` | Anonymous daily visit counters (hashed visitor id) |

## Design notes

- **Money is stored as integers** (`*_minor`, kopecks) to avoid floating-point errors; only the item price on a deal is still a float, which is a known inconsistency worth migrating.
- **The ledger explains the balance**: `ledger_entries.balance_after_minor` lets an admin audit endpoint detect any mismatch between `users.balance_minor` and the journal.
- **Escrow is two-sided**: `deals.payment_id` points to the current payment, `payments.deal_id` points back (the code comment calls it the *current* escrow payment, so a deal can be re-paid).
- **Soft links trade integrity for flexibility**: usernames and ids are used as logical keys in several places; adding real foreign keys is a candidate for the next schema iteration.
