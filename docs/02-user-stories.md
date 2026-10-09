# User stories

Format: *As a ⟨role⟩ I want ⟨goal⟩ so that ⟨value⟩*, with acceptance criteria. Stories are grouped into epics and tagged with the API area that implements them (see [API overview](05-api-overview.md)). **MVP** = implemented in the current snapshot.

## Epic A. Account and trust

**A1 · Register and verify e-mail** (MVP, `Auth`)
As a visitor I want to register and confirm my e-mail so that I can publish lots and trust other members.
- Given I submit a unique username, e-mail and password, then an account is created and a confirmation link is sent.
- Given my e-mail is not verified, when I try to publish a lot, then the action is refused with a clear message.
- E-mail is stored hashed for lookup and encrypted at rest.

**A2 · Reset my password** (MVP, `Auth`)
As a user I want to reset a forgotten password by e-mail so that I do not lose my account.
- A reset link is sent by e-mail; after the change I receive a "password changed" notification.

**A3 · Give legal consent** (MVP, `Auth`)
As the operator I want each user's acceptance of the terms and privacy policy recorded with the document version so that I can prove consent (152-FZ).

**A4 · Build reputation** (MVP, `Reviews`)
As a user I want to leave and read reviews so that members can judge whom to trust.
- One review per reviewer, reviewed user and lot; a user cannot review themselves.
- After a deal is completed the buyer is invited to leave a review.

## Epic B. Lots (supply side)

**B1 · Publish a trip lot** (MVP, `Lots`)
As a traveler I want to publish my route, arrival date, free weight and commission so that buyers can order items through me.
- The lot starts as `pending` and is invisible in the catalog until approved.

**B2 · Prove the trip** (MVP, `Lots`)
As a traveler I want to attach trip tickets to a lot so that moderators and buyers see the trip is real.

**B3 · Moderate lots** (MVP, `Lot moderation`)
As a moderator I want to see pending lots with the owner's statistics and approve or reject them so that fake or risky offers do not reach buyers.

**B4 · Find a lot** (MVP, `Lots`)
As a buyer I want to filter lots by route (country, city), category, dates and minimum weight, search by text, sort, and see lots on a map, so that I find a traveler on my route.

**B5 · Save a route subscription** (MVP: storage only, `Route alerts`)
As a buyer I want to save a route, date range and minimum weight so that my search needs are kept in my account.
- Subscriptions can be created, listed and deleted; duplicates and an excessive number per user are rejected.
- Notification delivery for matching lots is **not implemented yet** (see backlog).

## Epic C. Deal and escrow (core value)

**C1 · Offer an item** (MVP, `Deals`)
As a seller (lot owner) I want to create a deal with item description, photo, price and weight, optionally for a specific buyer, so that the buyer sees an exact final price.
- Final price = item price × (1 + lot commission %).
- Requested weight cannot exceed the lot's free weight.

**C2 · Pay safely** (MVP, `Deals`, `Payments`)
As a buyer I want my money to be held by the payment provider, not handed to a stranger, so that I only pay for what I actually receive.
- After a successful hold the deal becomes `paid`, the weight is deducted from the lot, both sides are notified.
- If the bank cancels the payment, I am notified and can retry.

**C3 · Prove the purchase** (MVP, `Deals`)
As a seller I want to upload up to five photos of the item so that the buyer can confirm it before the hand-over.

**C4 · Confirm and release** (MVP, `Deals`)
As a buyer I want to confirm the photos and later confirm receipt so that the money is released to the seller only when I am satisfied.
- Only the buyer can complete the deal, and only after the photos are confirmed.
- On completion the seller is paid minus the platform commission.

**C5 · Signal arrival** (MVP, `Deals`)
As a seller I want a one-tap "I have arrived" so that the buyer is told to arrange the meeting.

**C6 · Cancel a deal** (MVP, `Deals`)
As a participant I want to cancel an open deal so that held money is refunded and the weight returns to the lot.

**C7 · Resolve a dispute** (MVP, `Deals`)
As a participant I want to open a dispute with a reason (after payment) so that a moderator decides fairly.
As a moderator I want to read the deal thread and the buyer-seller chat history and rule for the seller (release) or the buyer (refund) so that conflicts end with a recorded decision.

**C8 · Handle a manual payment proof** (MVP, `Deals`)
As a moderator I want to confirm or reject an uploaded payment proof so that the manual payment flow stays safe when the automatic one is off.

## Epic D. Communication

**D1 · Chat with a counterpart** (MVP, `Messages`)
As a user I want a direct chat and a per-deal chat so that we can agree on the hand-over without leaving the platform.

**D2 · Get notified** (MVP, `Notifications`)
As a user I want in-app notifications for key deal events so that I do not miss a step.

**D3 · Contact support inside a deal** (MVP, `Deals`)
As a participant I want to call support from a deal and see the moderator's replies in the same thread.

## Epic E. Money out

**E1 · Save payout details** (MVP, `Payments`)
As a seller I want to save a bank card or an SBP phone number so that I can be paid. Raw details are tokenized; staff never see the token.

**E2 · Withdraw funds** (MVP, `Wallet`)
As a seller I want to request a withdrawal with limits and a cooldown so that I control when money leaves the platform.
- Requests may require manual staff review; every balance change appears in my ledger.

**E3 · Audit the money** (MVP, `Admin`)
As an admin I want a ledger and an audit check so that any mismatch between balances and the journal is detected, and manual adjustments always carry the author and a comment.

## Epic F. Administration

**F1 · Manage roles** (MVP, `Admin`)
As an admin I want to promote or demote users so that moderation is staffed without touching the database.

**F2 · Remove abusive reviews** (MVP, `Admin`)
As an admin I want to delete a review so that harassment does not stay on a profile.

## Backlog ideas (not implemented)

- Notify subscribers when a lot matching their route subscription is approved.
- Real foreign keys for `buyer_username` and chat links; move deal prices to integer minor units.
- Automatic escalation of deals stuck in `photos_uploaded` for N days.
- Seller rating weighted by deal value; badge for verified travelers.
- Customs and prohibited-goods checklist shown before a deal is created.
