# Veilmark proposal

## Track and stage

**Track:** Age / Eligibility Gate  
**Stage:** self-reported prototype  
**Evidence status:** implementation documentation only; deployment and submission evidence are outstanding.

Veilmark explores whether a person can satisfy a narrowly defined eligibility rule without publishing the value used to satisfy it. The prototype is intentionally modest: the member enters a self-reported signal, a Compact circuit checks it against a public threshold, and the contract records a passport-scoped nullifier receipt. It is not real age verification, credential authentication, personhood, or Sybil resistance.

## Problem

A gate often needs only a yes/no eligibility outcome, but conventional checks ask for more: a document image, a source account, a wallet address, or a retained identity record. Even when the operator does not need those details, the check can create a durable identity trail.

Age and eligibility gates are a useful privacy test case because the decision can be expressed as a predicate—“the supplied value meets this threshold”—while the value and its source remain sensitive. Veilmark focuses on that predicate and makes the limits visible rather than presenting a self-report as a credential.

## Proposed product

An operator creates a public signal window with:

- a numeric eligibility threshold;
- a passport that scopes replay protection;
- a future expiry timestamp;
- an issuer tag for public window metadata;
- a capacity; and
- a commitment to an operator secret.

A member connects a Midnight-compatible browser wallet, supplies a private signal and private phrase, and invokes `mark_passed`. The circuit checks that the window is live, not expired, and not full; checks the hidden signal against the public threshold; derives a domain-separated receipt nullifier from the phrase and passport; and rejects a receipt that has already been spent. Only the public rule and the anonymous receipt state are written to the ledger.

The operator console supports browser deployment on Preview or Preprod, loading an existing address, and the three lifecycle actions:

- **pause**: stop new member receipts;
- **resume**: reopen a paused window; and
- **rotate**: publish a new threshold, passport, expiry, issuer tag, and capacity, then make the window live.

The operator secret is used as a private witness and is checked against the public commitment. It is never intended to be a public circuit argument.

## Privacy model

| Information | Public ledger | Private witness or local state |
| --- | --- | --- |
| Threshold, passport, expiry, issuer tag, live state, capacity | Yes | No |
| Accepted count | Yes | No |
| Receipt nullifier and receipt-to-passport index | Yes | No |
| Member signal | No | Yes, supplied by the browser witness |
| Member phrase | No | Yes, used to derive the receipt |
| Operator secret | No | Yes, used for operator authorization |
| Source credential | Not accepted by this prototype | Would require a future credential adapter |

A receipt is a replay guard, not a credential and not an identity. The same phrase in the same passport produces a repeatable public nullifier and is rejected after use. A different phrase can produce a different receipt, and the contract does not prove that different phrases belong to different people.

## Explicit limitations and threat model

### No real credential authentication

The browser's signal field is editable. A user can enter any value and, if it clears the threshold, the proof says only that this value cleared the rule. There is no issuer signature, document verification, revocation check, age attestation, or trusted source binding. The `issuer_tag` is public window metadata; it does not authenticate an issuer.

### No Sybil resistance

The prototype does not establish one person per receipt, one wallet per person, or one identity per passport. A member can choose another phrase, wallet, or self-reported value. Passport-scoped nullifiers prevent accidental reuse of one phrase in one window; they do not prevent deliberate identity multiplication.

### Remote prover trust

Proof generation is performed through the connected wallet/prover path. The ZK circuit is intended not to reveal the witness values on-chain, but this prototype does not make the prover service trustless. Operators and members must account for that service's availability, metadata handling, and wallet policies. Transaction timing, circuit choice, public state changes, and other network-level observations remain possible.

### Operator secret custody

The browser holds the operator secret in memory for the session and asks the operator to back it up manually. Losing it prevents recovery of operator actions. Compromising it permits anyone who has it to authorize actions for that deployment. This prototype does not provide a key-recovery or rotation-of-operator-key mechanism.

## Why Midnight and Compact

Veilmark needs a public rule and public replay state alongside a private comparison input. Compact makes those boundaries explicit through ledger declarations, witnesses, circuits, and deliberate disclosure of public writes. Midnight can validate the predicate and update the public receipt set without requiring the member's signal or phrase to become ledger data.

The browser wallet path is also part of the product boundary: the wallet supplies the selected network session, proving path, transaction approval, and balancing flow. The application supports browser-led deployment and operation on Preview or Preprod; this proposal does not claim that either network has been deployed or verified for Veilmark yet.

## Implementation map

```text
contracts/veilmark.compact
  ├── public window configuration and receipt state
  ├── private signal, phrase, and operator-secret witnesses
  ├── mark_passed
  ├── pause_window / resume_window / rotate_window
  └── domain-separated pure receipt and operator commitments

contracts/managed/veilmark
  └── generated contract bindings and proving assets

frontend/ (workspace: veilmark-frontend)
  ├── /gate  member self-reported signal flow
  ├── /admin browser deploy and operator lifecycle controls
  ├── /field public indexed state
  └── /privacy privacy boundary and limitations
```

The Compact source of truth is `contracts/veilmark.compact`; generated output is expected under `contracts/managed/veilmark`. The frontend consumes the generated bindings and browser-served proving assets. The repository has no separate master guide; the operational procedure is in `docs/setup.md` and the evidence plan is in `docs/submission-checklist.md`.

## Acceptance criteria for the prototype

The implementation is ready for review only when the owner supplies fresh evidence for each item below:

1. `contracts/veilmark.compact` compiles and generated assets are synchronized into the frontend.
2. A fresh contract test run demonstrates threshold rejection, receipt replay rejection, operator authorization, pause/resume, and receipt domain separation.
3. A browser wallet can connect on the selected Preview or Preprod network.
4. The `/admin` flow can deploy a window and report indexer confirmation, with the address and transaction captured by the owner.
5. The `/gate` flow can submit a qualifying self-reported signal and show the indexed anonymous receipt.
6. Pause, resume, and rotate actions are exercised through the operator console.
7. A current CI run shows install, compile, tests, type-check, and build passing.
8. Submission materials include screenshots, a live hosted surface, a video, a product X presence/post, approval evidence, and the required repository/commit history.

None of these acceptance items is marked complete by this proposal. In particular, no compile, test, deployment, live-hosting, video, product X, approval, or CI result is claimed until the owner verifies and records it.

## Level 1–4 evidence plan

All items are outstanding. Commit counts are cumulative targets, not claims about the current repository.

### Level 1 — New Moon

- [ ] Public repository identified and submission-ready.
- [ ] At least 5 meaningful commits.
- [ ] Fresh `npm run compile` output showing Veilmark circuits and generated `contracts/managed/veilmark` assets.
- [ ] Browser deployment to Preview or Preprod.
- [ ] Screenshot and recorded address/transaction evidence for that deployment.
- [ ] Setup and public/private-boundary documentation.

### Level 2 — Waxing Crescent

- [ ] Matching Preview/Preprod wallet connection shown in the browser.
- [ ] Qualifying self-reported proof and indexed nullifier receipt shown.
- [ ] Live hosted frontend URL.
- [ ] Short wallet/proof video.
- [ ] At least 8 meaningful commits.

### Level 3 — First Quarter

- [ ] Screenshot or log of at least 3 passing tests from a fresh run.
- [ ] Indexed public-field evidence for threshold, lifecycle, count, and receipts.
- [ ] Passing CI run covering install, compile, tests, type-check, and build.
- [ ] Track approval evidence.
- [ ] At least 10 meaningful commits.

### Level 4 — Waxing Gibbous

- [ ] Live MVP on Preview or Preprod with browser deployment documented.
- [ ] Stable live hosting.
- [ ] One-minute (or equivalent short) product walkthrough video.
- [ ] Product X profile/post.
- [ ] Final UI, deployment, test, and CI screenshots.
- [ ] Final approval evidence.
- [ ] At least 15 meaningful commits.

The owner must complete the deployments, screenshots, public repository and commit record, live hosting, video, product X presence/post, approval, and CI passing runs. Veilmark remains a self-reported prototype until those limitations and evidence are addressed.

## Roadmap after the prototype

1. Replace the editable signal with a verifiable credential or attestation adapter while keeping the witness private.
2. Define issuer registration, credential expiry, revocation, and disclosure policies.
3. Specify a genuine uniqueness or Sybil-resistance model instead of implying that a nullifier provides one.
4. Review key custody, prover trust, metadata leakage, and lifecycle semantics before any real age or eligibility use.
5. Re-run the full Preview/Preprod deployment and submission evidence process after each material security change.
