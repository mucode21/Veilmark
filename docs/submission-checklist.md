# Veilmark submission checklist

**Track:** Age / Eligibility Gate  
**Stage:** self-reported prototype  
**Current evidence status:** outstanding  
**Deployment status:** no Preview or Preprod deployment is verified in this repository.

Every item below is intentionally unchecked. The owner must perform the setup, collect fresh evidence, and update this file only after the evidence exists. Existing source files or generated artifacts are not a substitute for a recorded compile, test, deployment, CI, or submission result.

There is no separate master guide. Follow [`setup.md`](setup.md) for commands and browser operation, then use this page as the evidence ledger.

## Product boundary to state in every submission

- Veilmark is a **self-reported** prototype for an Age / Eligibility Gate.
- The private signal is editable and is not backed by real credential authentication, issuer signatures, age attestation, or document verification.
- The contract provides receipt replay protection for a phrase within a passport, but it provides **no Sybil resistance** or proof of unique personhood.
- Proof generation uses the connected wallet/prover path; remote prover trust and metadata handling remain explicit assumptions.
- The implementation supports browser deployment and operation on Preview or Preprod, but no network deployment is verified until the owner captures indexed evidence.

## Evidence record

Complete these fields with current evidence. Do not paste an invented, stale, or unverified address.

```text
Repository: https://github.com/mucode21/Veilmark
Repository owner: mucode21
Track: Age / Eligibility Gate
Selected network (Preview or Preprod): Preprod
Contract address: 87cbdde748290bd2ad16c8868c140c3122b25041dd4e896605b2638051a0e5a1
Deployment transaction: 88aaedeb1c680f955c4e3185ce410c2d8c09457bed9b4c5ef7c4319fb88a6420
Explorer contract URL: https://explorer.1am.xyz/contract/87cbdde748290bd2ad16c8868c140c3122b25041dd4e896605b2638051a0e5a1
Explorer transaction URL: https://explorer.1am.xyz/tx/88aaedeb1c680f955c4e3185ce410c2d8c09457bed9b4c5ef7c4319fb88a6420?network=preprod
Live hosted frontend: https://veilmarkmidnight.netlify.app/
Demo video: https://drive.google.com/file/d/10fjHC8MD2z2g_9uBrjmKYx-1uCfIXN9D/view?usp=sharing
```

## Required setup and verification evidence

### Toolchain and generated artifacts

- [ ] Node.js 22+ version captured.
- [ ] npm 10+ version captured.
- [ ] Compact compiler 0.31.0 version captured.
- [ ] `npm install` completed from the repository root.
- [ ] `npm run compile` completed from the repository root.
- [ ] Compile output identifies `contracts/veilmark.compact` as the source.
- [ ] Generated output is present under `contracts/managed/veilmark`.
- [ ] Frontend bindings and browser proving assets are synchronized under `frontend/src/managed/contract` and `frontend/public/managed`.
- [ ] Screenshot or log shows the expected Veilmark circuits/assets.

### Local checks

- [ ] `npm test` completed and its output is attached.
- [ ] At least 3 tests are visibly passing in the attached output.
- [ ] `npm run typecheck` completed and its output is attached.
- [ ] `npm run build` completed and its output is attached.
- [ ] `npm run check` completed, or the equivalent individual outputs are attached.
- [ ] No command is marked passing based only on an old screenshot or a generated directory.

### Browser and network checks

- [ ] A compatible injected browser wallet is shown as detected.
- [ ] The wallet connects on the same Preview or Preprod network selected by the app.
- [ ] A browser deployment is submitted from `/admin`.
- [ ] The indexed deployment address and transaction are captured.
- [ ] `/gate` creates a qualifying self-reported proof and the anonymous receipt is indexed.
- [ ] `/admin` pause action is submitted and indexed.
- [ ] `/admin` resume action is submitted and indexed.
- [ ] `/admin` rotate action is submitted and indexed.
- [ ] Public field evidence shows threshold, passport/window metadata, expiry, lifecycle, accepted count, capacity, and receipt state.

### Submission materials

- [ ] Public repository is identified and accessible to reviewers.
- [ ] Required repository screenshots are captured: compile, deployment, tests, UI/public field, and CI.
- [ ] Required commit history is present and the count is recorded.
- [ ] Live hosting is available and its URL is recorded.
- [ ] Demo video is recorded and its URL is recorded.
- [ ] Product X profile/post is created and its URL is recorded.
- [ ] Track approval evidence is recorded.
- [ ] CI has a passing run for install, compile, tests, frontend type-check, and build, and the run URL/screenshot is recorded.

## Level 1–4 exact outstanding evidence

The requirements are cumulative. Every checkbox remains outstanding until the owner supplies the corresponding artifact.

### Level 1 — New Moon

**Setup/code evidence**

- [ ] `contracts/veilmark.compact` is compiled with the expected Compact toolchain.
- [ ] Generated bindings, keys, and ZK artifacts are present under `contracts/managed/veilmark` and synchronized into the workspace frontend.
- [ ] The README, proposal, and setup explain the private/public boundary and the prototype limitations.

**Owner evidence**

- [ ] Public repository identified for submission.
- [ ] At least **5 meaningful commits**.
- [ ] Fresh compile screenshot/log with Veilmark circuit names.
- [ ] Browser deployment to Preview or Preprod.
- [ ] Deployment screenshot showing the network and indexed contract address.
- [ ] Deployment transaction evidence recorded.

### Level 2 — Waxing Crescent

**Feature evidence**

- [ ] Matching Preview or Preprod wallet connection shown in the browser.
- [ ] `/gate` threshold preflight shown with a self-reported signal.
- [ ] Successful proof submission shown.
- [ ] Indexed anonymous receipt/nullifier shown.
- [ ] Pause, resume, or rotate operator flow shown in the browser.

**Owner evidence**

- [ ] Live hosted frontend URL recorded.
- [ ] Short wallet/proof demo video recorded and linked.
- [ ] At least **8 meaningful commits**.

### Level 3 — First Quarter

**Verification evidence**

- [ ] Screenshot/log of **at least 3 passing tests** from a fresh run.
- [ ] Public field screenshot/log showing rule, lifecycle, count, and receipt behavior.
- [ ] Type-check and frontend build evidence.
- [ ] CI run passing install, compile, tests, type-check, and build.

**Owner evidence**

- [ ] Passing CI workflow URL and screenshot recorded.
- [ ] Track approval evidence recorded.
- [ ] At least **10 meaningful commits**.

### Level 4 — Waxing Gibbous

**Product evidence**

- [ ] Live MVP hosted for reviewers.
- [ ] Browser Preview/Preprod deployment procedure is reproducible from `docs/setup.md`.
- [ ] Final UI screenshots include member gate, operator console, and public field.
- [ ] Final deployment screenshot includes indexed network/address/transaction evidence.
- [ ] Final test and CI screenshots are current.

**Owner evidence**

- [ ] Live hosting URL recorded.
- [ ] One-minute or equivalent product walkthrough video recorded and linked.
- [ ] Product X profile/post recorded and linked.
- [ ] Final approval evidence recorded.
- [ ] At least **15 meaningful commits**.

## Evidence rules

1. A wallet-submitted transaction is not an indexed deployment. Record both submission and indexer confirmation.
2. A local Docker run is not Preview/Preprod evidence.
3. A generated managed directory is not proof that the current source compiled; attach a fresh compile output.
4. A test screenshot without its command, repository revision, and date is incomplete evidence.
5. Do not describe the editable signal as authenticated age, credential ownership, or personhood.
6. Do not describe a receipt nullifier as Sybil resistance.
7. State the remote prover trust assumption wherever proof privacy is described.
8. Do not add a deployment address, live URL, video URL, product X URL, approval claim, or CI claim until the owner verifies it.

## Final owner sign-off

- [ ] I performed the deployment and captured indexed Preview/Preprod evidence.
- [ ] I captured current compile, test, type-check/build, UI, and CI screenshots/logs.
- [ ] I initialized or identified the public repository and recorded the required meaningful commits.
- [ ] I published live hosting, the demo video, and the product X presence/post.
- [ ] I obtained and recorded approval for the Age / Eligibility Gate track.
- [ ] I have described Veilmark as a self-reported prototype, not as credential authentication or Sybil resistance.
