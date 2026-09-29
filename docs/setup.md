# Veilmark setup

This is the repository setup reference for Veilmark's **Age / Eligibility Gate** track. Veilmark is a self-reported prototype: the member signal is editable, there is no credential authentication or Sybil resistance, and a wallet/prover service is trusted for proof generation.

There is no separate master guide in this repository. This page covers local preparation and the browser deployment path; [`submission-checklist.md`](submission-checklist.md) lists the evidence the owner still has to produce.

## 1. Repository and toolchain

Run commands from the repository root:

```bash
node --version       # Node.js 22 or newer
npm --version        # npm 10 or newer
compactc --version   # Compact compiler 0.31.0, or the supported compact command
```

The project uses npm workspaces. The browser package is named `veilmark-frontend`.

Required for the complete flow:

- Node.js 22+ and npm 10+;
- Compact compiler 0.31.0 available on `PATH`, or the supported WSL installation on Windows;
- a Midnight-compatible injected browser wallet, such as 1AM or Lace, for Preview or Preprod;
- enough network funds for the wallet's proving, balancing, and transaction flow;
- Docker Desktop only if the local stack is needed.

On Windows, `scripts/compile-contract.mjs` tries the local compiler, supported WSL paths, and the `compact compile` command. If none is installed, compilation cannot be treated as verified.

## 2. Install dependencies

```bash
npm install
```

The root package owns the Compact/runtime tests and the workspace scripts. Do not install a second frontend package in a different directory; use the existing workspace:

```bash
npm run dev -w veilmark-frontend
npm run build -w veilmark-frontend
```

## 3. Environment configuration

Copy `frontend/.env.example` to `frontend/.env` when build-time values are needed. Keep secrets out of committed environment files.

The example supports:

- `VITE_NETWORK_ID` (`preprod` by default in the example);
- a network-specific or generic contract address after the owner has deployed one; and
- network-specific indexer URLs when defaults are not suitable.

No contract address is supplied by this guide. The operator console can save a deployed or loaded address in browser local storage for the selected network. Preview and Preprod addresses must never be mixed.

## 4. Compile and synchronize generated assets

The Compact source of truth is:

```text
contracts/veilmark.compact
```

Run:

```bash
npm run compile
```

The root compile script:

1. invokes the Compact compiler;
2. writes generated output to `contracts/managed/veilmark`;
3. runs the repository runtime compatibility hook; and
4. copies generated contract bindings and browser assets into the frontend.

The browser expects these generated destinations:

```text
contracts/managed/veilmark/contract/
contracts/managed/veilmark/keys/
contracts/managed/veilmark/zkir/
frontend/src/managed/contract/
frontend/public/managed/
```

The operator console loads assets from `/managed`. Before a browser deployment, check that the build output includes compiler metadata, the bindings, and the prover/verifier/ZK artifacts for `mark_passed`, `pause_window`, `resume_window`, and `rotate_window`.

For a quick development iteration, the Compact toolchain may support:

```bash
compact compile --skip-zk contracts/veilmark.compact contracts/managed/veilmark
```

This is not a substitute for a full generated-asset compile or deployment evidence. Do not mark compilation complete without capturing the actual command output.

## 5. Local verification commands

The repository provides these commands:

```bash
npm test
npm run typecheck
npm run build
npm run check
```

`npm run check` runs the root test, frontend type-check, and frontend build sequence. A fresh run is required before reporting any command as passing. This documentation does not claim that compile, tests, type-check, build, or CI have passed.

The deterministic tests exercise the Compact contract through generated bindings. They cover the private threshold predicate, receipt replay behavior, operator authorization, pause/resume, and receipt domain separation. They do not prove real credential authentication, age verification, personhood, or Sybil resistance.

## 6. Optional local network

Start and stop the local Docker stack with:

```bash
npm run env:up
npm run env:down
```

A typical local preparation is:

```bash
npm run env:up
npm run compile
npm test
npm run env:down
```

The local stack is not the Preview/Preprod deployment path. A local test or local address must not be presented as network deployment evidence.

## 7. Start the frontend

```bash
npm run dev -w veilmark-frontend
```

Use the Vite URL and these routes:

| Route | Purpose |
| --- | --- |
| `/admin` | Connect a wallet, deploy/load a window, pause, resume, and rotate |
| `/gate` | Submit a self-reported private signal and phrase |
| `/field` | Read indexed public window state |
| `/privacy` | Explain the public/private boundary and limitations |

The default network is Preprod unless the frontend configuration or operator selection chooses Preview. The wallet network and selected application network must match.

## 8. Browser wallet connection

1. Install a compatible Midnight wallet extension and make sure it exposes an injected wallet to the browser.
2. Open `/admin`.
3. Select `Preview` or `Preprod` in the network selector.
4. Select the intended injected wallet if more than one is available.
5. Connect on that exact network.
6. Wait for wallet synchronization before submitting a proof or deployment.

The application rejects a session whose wallet network differs from the selected network. Disconnect or change the wallet network before retrying; do not use a contract address from another network.

## 9. Browser deployment on Preview or Preprod

The operator is responsible for this flow and for recording its evidence:

1. On `/admin`, select the target network and connect the matching wallet.
2. Enter a 64-hex operator secret. Back it up offline before continuing. The browser stores it only in memory for the session; losing it is a manual recovery problem.
3. Enter a whole-number threshold, capacity, and future Unix timestamp. The current UI limits threshold and capacity to its documented form bounds.
4. Confirm that the operator secret was backed up.
5. Select **Deploy new window**.
6. Approve wallet connection, proving, balancing, and transaction submission.
7. Record the network, contract address, and transaction identifier returned by the wallet.
8. Wait for the selected network's public indexer to report the contract. The UI distinguishes a submitted transaction from an indexed deployment.
9. Capture screenshots of the configuration, wallet/network match, submitted transaction, indexed address, and public state.
10. Keep the address private until the owner decides what submission evidence is appropriate; this guide intentionally contains no deployment address.

The constructor arguments are passed in this order:

```text
(threshold, passport, deadline, issuer, operator_hash, limit)
```

The browser generates fresh passport and issuer values for a new deployment and derives `operator_hash` from the supplied operator secret. The source credential is not part of deployment because this is a self-reported prototype.

## 10. Member proof and receipt flow

After an operator has loaded an indexed address:

1. Open `/gate` on the same selected network.
2. Connect a matching browser wallet.
3. Read the public threshold, expiry, live state, and capacity.
4. Enter a self-reported signal that meets the displayed threshold.
5. The browser derives or loads a local phrase for the receipt witness.
6. Choose **Create private receipt** and approve the wallet transaction.
7. Wait for the public indexer to show the new anonymous receipt.

The contract receives no public arguments for `mark_passed`. The signal and phrase are witness inputs. The public result is the receipt nullifier and updated accepted state. The same phrase for the same passport is rejected after use.

The browser/prover path is trusted for this prototype. The proof is not a credential check, and the receipt is not Sybil resistance.

## 11. Operator lifecycle actions

On `/admin`, load the indexed contract for the selected network and enter the same operator secret used at deployment:

- **Pause window** calls `pause_window` and sets `live` to false.
- **Resume window** calls `resume_window` and sets `live` to true.
- **Rotate window** calls `rotate_window` with a new threshold, passport, expiry, issuer tag, and capacity, then makes the window live.

Each action must be approved by the wallet and confirmed by observing the changed indexed public state. Rotation does not erase prior accepted-count or receipt state in the current contract implementation.

## 12. Troubleshooting without overstating evidence

- **Compiler not found:** install Compact 0.31.0 or configure the supported WSL path, then rerun `npm run compile`.
- **Wallet not detected:** unlock the extension, check that it exposes an injected API, reload the page, and confirm the wallet is on Preview or Preprod.
- **Network mismatch:** select the same network in `/admin` and in the wallet. A Preview address cannot be used as a Preprod address.
- **Assets missing at `/managed`:** run `npm run compile` and verify the generated copies under `frontend/public/managed`.
- **Deployment says submitted but not indexed:** retain the transaction identifier and wait for the selected network's indexer; do not record it as a verified deployment until indexed evidence is captured.
- **Proof waits or fails:** check wallet synchronization, funds, network selection, and prover availability. A successful local setup does not prove remote network availability.
- **Operator action fails:** re-enter the original 64-hex operator secret. The contract checks the commitment, and this prototype has no secret recovery flow.

## 13. Evidence handoff

Before calling setup complete, the owner must attach fresh evidence for:

- compile output and generated circuit assets;
- at least 3 passing tests;
- type-check and frontend build;
- Preview or Preprod browser deployment, address, and transaction;
- wallet-connected member proof and indexed receipt;
- pause, resume, and rotate actions;
- a passing CI run that covers install, compile, tests, type-check, and build;
- live hosting, a demo video, product X presence/post, approval, repository identity, and required commit history.

Use [`docs/submission-checklist.md`](submission-checklist.md) to record those artifacts. Until captured, all of them remain outstanding.
