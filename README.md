# Veilmark

> A private, self-reported eligibility threshold gate for Midnight.

**Track:** Age / Eligibility Gate  
**Stage:** self-reported prototype  
**Network path:** Midnight Preprod Network  
**Deployment status:** Live on Preprod  
**Live dApp:** [https://veilmarkmidnight.netlify.app/](https://veilmarkmidnight.netlify.app/)  
**Contract address:** [`87cbdde748290bd2ad16c8868c140c3122b25041dd4e896605b2638051a0e5a1`](https://explorer.1am.xyz/contract/87cbdde748290bd2ad16c8868c140c3122b25041dd4e896605b2638051a0e5a1)  
**Deployment transaction:** [`88aaedeb1c680f955c4e3185ce410c2d8c09457bed9b4c5ef7c4319fb88a6420`](https://explorer.1am.xyz/tx/88aaedeb1c680f955c4e3185ce410c2d8c09457bed9b4c5ef7c4319fb88a6420?network=preprod)  
**Demo Video:** [Watch Walkthrough Demo](https://drive.google.com/file/d/10fjHC8MD2z2g_9uBrjmKYx-1uCfIXN9D/view?usp=sharing)

Veilmark demonstrates a narrow privacy boundary: an operator publishes an eligibility threshold, and a member proves that a locally supplied signal meets that threshold without putting the signal or the member's phrase on the public ledger. A successful proof records an anonymous receipt nullifier. The result is a prototype of a private gate, not an age-verification service or an identity system.

## Initial product idea

Veilmark is a privacy-first eligibility window for communities that need a yes/no threshold check without collecting the underlying value, such as an early-access room, a research cohort, or an age-gated experience. The first prototype makes the boundary explicit with a self-reported signal today, leaving room for a verifiable credential adapter later.

## What the prototype does

- An operator deploys a signal window with a threshold, passport, expiry, issuer tag, operator commitment, and capacity.
- A member supplies a **self-reported** private signal and a private 32-byte phrase in the browser.
- `mark_passed` checks the signal against the public threshold and records a passport-scoped receipt nullifier when the check succeeds.
- The operator can pause, resume, or rotate the public window using the private operator secret that matches the on-chain commitment.
- The browser operator console can deploy and operate the contract on **Preview** or **Preprod**, provided the connected wallet is on the selected network.
- The public field reads the intentionally public ledger state; it does not turn the private witness into an identity record.

### Honest limitations

The signal is editable and self-reported. Veilmark does **not** authenticate a credential, verify a government document, prove a person's age, or establish that the signal came from an issuer. It also provides **no Sybil resistance**: a person can choose another phrase or another self-reported value. The receipt prevents reuse of the same phrase in the same passport; it is not proof of unique personhood.

Proof generation is delegated to the connected wallet/prover service. The circuit keeps witness values out of the public ledger, but this prototype still requires trust in that service and in the wallet's metadata and transaction policies. This is not a claim of production-grade privacy or authentication.

## Public/private boundary

| Data | Visibility | Purpose |
| --- | --- | --- |
| `signal_threshold`, `passport`, `window_end`, `issuer_tag`, `live`, `accepted`, `capacity` | Public ledger | Define the current window and its aggregate usage |
| `operator_commitment` | Public ledger | Bind operator actions to a secret without storing that secret |
| `spent_receipts` and `receipt_index` | Public ledger | Reject a reused receipt and expose the anonymous outcome |
| Private signal | Witness | Compare the member's self-report with the threshold |
| Member phrase | Witness | Derive the one-time receipt preimage |
| Operator secret | Witness | Authorize pause, resume, and rotation |

`make_receipt_nullifier` uses a domain-separated hash of the phrase and passport. Reusing a phrase in one passport therefore derives the same public receipt and is rejected; changing the passport changes the receipt domain. This is replay protection, not an identity claim.

## Contract circuits

The Compact source of truth is [`contracts/veilmark.compact`](contracts/veilmark.compact). The generated contract and proving assets belong under `contracts/managed/veilmark`.

| Circuit | Role | Public arguments | Private witnesses |
| --- | --- | --- | --- |
| `mark_passed` | Check the hidden signal, enforce the live/expiry/capacity rules, and record a receipt | none | signal, phrase |
| `pause_window` | Stop new receipts | none | operator secret |
| `resume_window` | Re-enable the window | none | operator secret |
| `rotate_window` | Replace threshold, passport, expiry, issuer tag, and capacity; make the window live | new configuration | operator secret |
| `operator_public_key` | Derive the operator commitment | secret input to a pure circuit | secret input |
| `make_receipt_nullifier` | Derive the passport-scoped receipt | phrase and passport to a pure circuit | phrase input |

Rotation changes the public configuration and sets `live` to true. It does not itself erase prior accepted-count or receipt state; operators should treat this as a prototype window state machine and verify the intended policy before relying on it.

## Repository layout

```text
contracts/veilmark.compact          Compact source of truth
contracts/managed/veilmark/         generated contract, keys, and ZK artifacts
frontend/                           npm workspace: veilmark-frontend
frontend/src/pages/GatePage.tsx     member signal gate
frontend/src/pages/AdminPage.tsx    browser deployment and operator console
frontend/src/pages/ObservatoryPage.tsx public ledger state view
frontend/src/managed/contract/      frontend copy of generated bindings
frontend/public/managed/             browser-served generated proving assets
src/test/                           deterministic contract and privacy tests
scripts/compile-contract.mjs        compile and artifact-generation entry point
docs/setup.md                       setup and browser deployment procedure
docs/submission-checklist.md        level 1–4 evidence checklist
```

Generated files under `contracts/managed/veilmark`, `frontend/src/managed/contract`, and `frontend/public/managed` should be regenerated rather than hand-edited.

## Requirements

- Node.js 22 or newer
- npm 10 or newer
- Compact compiler 0.31.0 available as `compactc`/`compact` or through the supported WSL path on Windows
- A Midnight-compatible injected browser wallet for Preview or Preprod, such as 1AM or Lace
- Network funds required by the selected wallet for proving and transaction submission
- Docker Desktop only when using the local development stack

The repository has no separate master guide. Use [`docs/setup.md`](docs/setup.md) for the complete local and browser setup, and [`docs/submission-checklist.md`](docs/submission-checklist.md) for the evidence still required from the owner.

## Install and verify locally

From the repository root:

```bash
npm install
npm run compile
npm test
npm run typecheck
npm run build
```

The `compile` script invokes the Compact compiler on `contracts/veilmark.compact`, writes generated output to `contracts/managed/veilmark`, applies the repository compatibility hook, and synchronizes browser assets. The frontend build is the `veilmark-frontend` npm workspace.

For the combined local check:

```bash
npm run check
```

These commands are instructions, not evidence. This README intentionally does not report a passing compile, test, type-check, build, or CI result. Capture fresh output before marking any verification item complete.

For a faster compiler iteration when supported by the installed toolchain:

```bash
compact compile --skip-zk contracts/veilmark.compact contracts/managed/veilmark
```

The skip-ZK path is a development aid only; it is not deployment evidence.

## Optional local network

The local stack is separate from the Preview/Preprod browser flow:

```bash
npm run env:up
npm run compile
npm test
npm run env:down
```

Do not interpret a local run as proof of a Preview or Preprod deployment. The default browser configuration targets Preprod unless the operator selects Preview in the console or configures the workspace environment.

## Run the browser frontend

```bash
npm run dev -w veilmark-frontend
```

Open the local URL printed by Vite, then use:

- `/admin` for wallet connection, deployment, loading an existing address, pause/resume, and rotation;
- `/gate` for the member proof flow;
- `/field` for public indexed state;
- `/privacy` for the in-app privacy explanation.

The frontend reads `frontend/.env` when present. Start from `frontend/.env.example` if build-time network or contract values are needed. A deployment can also be loaded into the browser through the operator console and stored in local storage; no deployment address is pre-populated in this documentation.

## Preview / Preprod browser deployment

The operator flow is deliberately browser-led:

1. Install and unlock a compatible Midnight browser wallet.
2. Start the frontend and open `/admin`.
3. Select **Preview** or **Preprod** and connect a wallet on that same network. A mismatched wallet session is rejected.
4. Enter a 64-hex operator secret and back it up offline. The UI keeps this secret in memory; recovery is manual.
5. Set the threshold, capacity, and future Unix expiry.
6. Choose **Deploy new window** and approve proving, balancing, and submission in the wallet.
7. Wait for the selected network's indexer to confirm the deployment. A submitted transaction is not the same as an indexed deployment.
8. Save the resulting contract address and network as owner evidence. Then use `/gate` to create a member receipt or the operator controls to pause, resume, and rotate.

The deployment constructor order is:

```text
(threshold, passport, deadline, issuer, operator_hash, limit)
```

The operator console loads proving assets from `/managed`. Before attempting a deployment, confirm that the generated compiler metadata, contract bindings, prover/verifier keys, and circuit artifacts are reachable from that path. The exact browser checklist is in [`docs/setup.md`](docs/setup.md).

### Live Preprod Deployment

Veilmark has been deployed and verified on the Midnight Preprod network:

- **Network:** Preprod
- **Live dApp URL:** [https://veilmarkmidnight.netlify.app/](https://veilmarkmidnight.netlify.app/)
- **Contract Address:** [`87cbdde748290bd2ad16c8868c140c3122b25041dd4e896605b2638051a0e5a1`](https://explorer.1am.xyz/contract/87cbdde748290bd2ad16c8868c140c3122b25041dd4e896605b2638051a0e5a1)
- **Deployment Transaction:** [`88aaedeb1c680f955c4e3185ce410c2d8c09457bed9b4c5ef7c4319fb88a6420`](https://explorer.1am.xyz/tx/88aaedeb1c680f955c4e3185ce410c2d8c09457bed9b4c5ef7c4319fb88a6420?network=preprod)
- **Demo Video Walkthrough:** [Google Drive Demo Video](https://drive.google.com/file/d/10fjHC8MD2z2g_9uBrjmKYx-1uCfIXN9D/view?usp=sharing)
- **Explorer:** [1AM Explorer](https://explorer.1am.xyz)

## What is and is not verified

At the time of this rewrite, this documentation makes no claim that:

- the Compact source has been compiled in the current environment;
- the test suite, type-check, frontend build, or CI workflow has passed;
- a contract has been deployed or indexed on Preview or Preprod;
- a live hosted frontend exists;
- a demo video, product X profile/post, or approval exists.

The owner must produce and attach those artifacts. The outstanding level-by-level evidence is recorded without completion marks in [`docs/submission-checklist.md`](docs/submission-checklist.md).

## Level 1–4 outstanding evidence

All items below are outstanding until the owner supplies current evidence. The levels describe submission maturity for this **Age / Eligibility Gate** self-reported prototype; they do not turn self-reported input into verified age or identity.

### Level 1 — New Moon

- [ ] Initialize or identify the public repository and record its canonical repository location.
- [ ] Make at least 5 meaningful commits.
- [ ] Run `npm run compile` and capture output showing the Veilmark circuits and generated `contracts/managed/veilmark` artifacts.
- [ ] Deploy from the browser operator console to Preview or Preprod and capture the network, contract address, and transaction evidence.
- [ ] Capture a deployment-address screenshot.
- [ ] Keep setup and public/private-boundary documentation available.

### Level 2 — Waxing Crescent

- [ ] Demonstrate a matching Preview or Preprod wallet connection in the browser.
- [ ] Demonstrate a successful threshold proof and newly indexed anonymous receipt.
- [ ] Publish a live hosted frontend URL.
- [ ] Record and publish a short wallet/proof walkthrough video.
- [ ] Make at least 8 meaningful commits.

### Level 3 — First Quarter

- [ ] Capture a screenshot or log of at least 3 passing tests from a fresh verified run.
- [ ] Capture the indexed public field showing rule, lifecycle, count, and receipt behavior.
- [ ] Provide a passing CI run for install, compile, tests, type-check, and build.
- [ ] Obtain and record track approval.
- [ ] Make at least 10 meaningful commits.

### Level 4 — Waxing Gibbous

- [ ] Operate a live MVP on Preview or Preprod with the browser deployment flow documented.
- [ ] Keep a live hosted product surface available.
- [ ] Publish the demo video and product X presence/post.
- [ ] Provide final UI, deployment, test, and CI screenshots.
- [ ] Provide final approval evidence.
- [ ] Make at least 15 meaningful commits.

The owner—not this README—must complete deployments, screenshots, repository/commit history, live hosting, video, product X presence, approval, and CI passing runs.

## Scope and non-goals

This repository demonstrates the privacy mechanics of a threshold gate. It does not include a real credential issuer, credential revocation, age attestation, personhood, Sybil resistance, production key management, or a claim that a remote prover is trustless. Those are explicit follow-up requirements before any real eligibility or age use.

## License

MIT. See [`LICENSE`](LICENSE).
