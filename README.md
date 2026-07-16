# Donnyoregon — Independent Web3 Security Researcher

## EVM & Move Bytecode Analyst

I specialize in mathematically complex DeFi protocols, low-level EVM/Move state corruption, and proprietary zero-day static analysis tooling. I find the critical vulnerabilities — missing bounds checks, unsafe truncations, protocol insolvency vectors — that top-tier auditing firms consistently miss.

I am available to audit your project. My tooling traces protocol insolvency bugs using manual bytecode reading and custom scanners that go deeper than basic address-level checks. Every report comes with bytecode-level verification and mainnet fork tests.

---

## Contact

| Channel | Address |
| :--- | :--- |
| Email | `clarkcorrin@gmail.com` |
| GitHub | [@donnyoregon](https://github.com/donnyoregon) |
| Cantina | `@arch` |

---

## Technical Stack

- **Languages:** Solidity, Yul, Move, Python, Rust, Bash
- **Frameworks:** Foundry (Forge/Cast/Anvil), Hardhat, Sui Move
- **Specialties:**
  - Custom EVM/Move bytecode vulnerability detection
  - On-chain forensics: storage slot diffing, bytecode decompilation, proxy implementation tracing
  - Forensic git monitoring and evidence preservation tooling

---

## Disclosure Record

The following cases are an immutable record of systemic fraud in the Web3 security industry. Each case includes the original vulnerability, the response, and preserved on-chain or repository evidence of the subsequent stealth fix.

---

### Case 1 — Marginal Protocol (Jan 2026)

| Field | Value |
| :--- | :--- |
| Submitted | Jan 31, 2026 |
| Platform | Cantina |
| CERT/CC | VU#643748 |
| Finding | #27 (Critical) |
| Vulnerability class | Unsafe downcast — unchecked `uint160` truncation of `sqrtPriceX96` |
| Impact | $100M debt settleable for ~$0.000000000000057005 ETH (99.999% loss) |
| Outcome | Rejected as duplicate of a previously rejected finding |

**Root cause.** `sqrtPriceX96` is a 256-bit value. The contract cast it to `uint160` via a raw bitwise `AND` with no bounds check. An attacker who can force `sqrtPriceX96` above `type(uint160).max` causes the pool to mis-price a settlement at effectively zero cost.

**The fraud.** The protocol paused on Feb 2. On Feb 4 (Block [24386649](https://etherscan.io/block/24386649), TX [`0xe021...89a`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/tx-proof/stealth_patch_tx.md)) it was silently patched. Cantina then rejected my Jan 31 report as a "duplicate" of an earlier finding — a finding that Cantina itself had previously rejected. A duplicate of a rejected finding is not a duplicate; it is unaddressed.

**Evidence archive ([donnyoregon/marginal-v1-disclosure](https://github.com/donnyoregon/marginal-v1-disclosure)):**

| Artifact | Link |
| :--- | :--- |
| Full forensic report | [`report-summary.md`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/report-summary.md) |
| Bytecode-level proof | [`bytecode_proof.md`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/bytecode-safe-cast/bytecode_proof.md) |
| Storage slot diff | [`slot_6_diff.md`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/storage-slot-diff/slot_6_diff.md) |
| Stealth patch TX proof | [`stealth_patch_tx.md`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/tx-proof/stealth_patch_tx.md) |
| Mainnet fork exploit | [`full_exploit_code.t.sol`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/full_exploit_code.t.sol) |
| Cantina rejection thread | [`Cantina_Triage_Rejection_Thread.pdf`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/cantina-rejection/Cantina_Triage_Rejection_Thread.pdf) |

---

### Case 2 — Frax Finance (Dec 2025)

| Field | Value |
| :--- | :--- |
| Discovered | Dec 2, 2025 |
| Submitted | Dec 5, 2025 |
| Platform | Internal (Frax team) |
| Contract | [`0xfDC69e6BE352BD5644C438302DE4E311AAD5565b`](https://etherscan.io/address/0xfDC69e6BE352BD5644C438302DE4E311AAD5565b) |
| Vulnerability class | Denial-of-service — zero-value redemption path |
| Error selector | `CannotRedeemZero()` — `0xb445ff79` |
| Outcome | Rejected as "impossible"; contract silently upgraded to include the guard |

**Root cause.** `FraxEtherRedemptionQueueV2` did not validate that the redemption amount was non-zero before processing. An attacker can submit a zero-amount redemption to trigger a revert that disrupts queue state, causing a denial-of-service on the redemption flow.

**The fraud.** My report was rejected internally with the response that the scenario was "impossible." The contract at `0xfDC69e6BE352BD5644C438302DE4E311AAD5565b` was subsequently upgraded to add the `CannotRedeemZero()` check — the exact guard my report identified as missing — with no disclosure and no payment.

**Evidence:** [Gist: Frax Ether Redemption Queue DoS](https://gist.github.com/donnyoregon/077da021e0a2534dc811cb7ab96b40e6)

---

### Case 3 — Walrus / MystenLabs (Dec 2025)

| Field | Value |
| :--- | :--- |
| Discovery period | Dec 2–4, 2025 |
| Repository | [MystenLabs/walrus](https://github.com/MystenLabs/walrus) |
| Preserved diff | [`DECEMBER_CODE_CHANGES.diff`](DECEMBER_CODE_CHANGES.diff) — 58,451 lines, 52 commits |
| Vulnerability class | Epoch-boundary desync; concurrent blob deletion race; event processor architectural flaw |
| Outcome | All fixes merged under `chore:`, `test:`, and docs commit labels with no CVE, no advisory, no disclosure |

The diff spans Dec 2 – Dec 30, 2025. The commits cluster in two distinct phases separated by a near-total silence in substantive node code after Dec 13. The pattern is: rapid covert stabilization of critical node-level bugs, followed by a flood of documentation commits that buries the fixes in noise.

#### Commit timeline

| Date | SHA | Label | Subject |
| :--- | :--- | :--- | :--- |
| Dec 2 | `6aba4f7` | `fix:` | Address race condition and deadlock in dropping `BlobRetirementNotify` |
| Dec 2 | `47e4c0a` | `feat:` | Add snapshot functionality to typed-store |
| Dec 2 | `da01e81` | `test:` | Fix `simtest_test_quorum_contract_upgrade` |
| Dec 2 | `7ad959b` | `chore:` | Address Move package build lint warning |
| Dec 2 | `371b2d3` | `ci:` | Update site-builder dependency jobs |
| Dec 4 | `a0fe265` | `chore:` | Refactor `BlobEventProcessor` — remove `SequentialProcessor` |
| Dec 5 | `7fa8129` | `fix:` | Avoid errors when storing metadata for untracked blobs |
| Dec 5 | `c246840` | `feat:` | Add price/capacity votes to committee info |
| Dec 8 | `092fa28` | `feat:` | Support targeted uploads and cooperative cancellation |
| Dec 8 | `165b051` | `fix:` | Delete per-object blob info when blob is deleted |
| Dec 9 | `c6e920a` | `chore:` | Remove `use_legacy_event_provider` option |
| Dec 9 | `bfe3f4c` | `docs:` | Update docs framework |
| Dec 10 | `5eb4ba4` | `fix:` | WAL-874 and WAL-876 — shutdown coordination |
| Dec 10 | `236d0ea` | `fix:` | Add metric to track recovery without deferral |
| Dec 10 | `c9af789` | `fix:` | Fix racing notification between catchup and checkpoint tailing |
| Dec 10 | `2755270` | `docs:` | Re-add static setup files removed in docs migration |
| Dec 10 | `2f05dc8` | `chore:` | Bump GitHub Actions dependencies |
| Dec 10 | `bb71eba` | `chore:` | Bump `js-yaml` |
| Dec 11 | `da15934` | `ci:` | Bump Sui testnet version to `testnet-v1.62.0` |
| Dec 11 | `749b9f4` | `docs:` | Fix redirects with Docusaurus plugin |
| Dec 11 | `fa1f023` | `docs:` | Fix `walrus.pdf` displaying in browser |
| Dec 11 | `356b9c9` | `docs:` | Objectives and use cases revision |
| Dec 11 | `4b47c19` | `chore:` | Remove single recovery symbol endpoint |
| Dec 11 | `f278d52` | `ci:` | Upgrade Slack API to v2 |
| Dec 11 | `b7e1e34` | `feat:` | Wire upload intent into client code |
| Dec 11 | `046ae12` | `docs:` | Fix redirects |
| Dec 11 | `beb12f4` | `chore:` | Bump Walrus version to `v1.40.0` |
| Dec 11 | `076f5f5` | `docs:` | Use `ws-resources` for redirects |
| Dec 11 | `26fdcc4` | `feat:` | Reading individual sliver respects aggregator max blob size |
| Dec 12 | `15654e2` | `fix:` | Use 1 SUI for gas budget in docs-site update |
| Dec 12 | `82ace2c` | `docs:` | Remove Docusaurus redirects; auto-generate HTML |
| Dec 13 | `0173958` | `test:` | Various test improvements |
| Dec 15 | `c480fd8` | `feat:` | Storage node list raw recovery symbols endpoint |
| Dec 15 | `55a124d` | `docs:` | Fix step loader plugin |
| Dec 15 | `a7453fe` | `docs:` | Style guide revisions to `dev-guide` and `operator-guide` |
| Dec 16 | `5de4901` | `docs:` | Fix broken link on homepage |
| Dec 16 | `9d1caac` | `docs:` | Fix redirects with `ws-resources.json` |
| Dec 16 | `3032f74` | `docs:` | Revisions to `/docs/usage` pages |
| Dec 17 | `e1163d2` | `docs:` | Update doc about list backup |
| Dec 18 | `d267da5` | `chore:` | Bump Rust to 1.92; fix new lint warnings |
| Dec 18 | `e45c0f3` | `docs:` | More Walrus design doc revisions |
| Dec 18 | `d9508d5` | `docs:` | Fix `docs/content` link |
| Dec 18 | `5bb3053` | `docs:` | More link fixes |
| Dec 18 | `af4ba5e` | `ci:` | Include `--ws-resources` argument in publish-docs workflow |
| Dec 18 | `e044d65` | `docs:` | Remove trailing comma |
| Dec 18 | `16a24d2` | `docs:` | Remove route for `design/future` |
| Dec 19 | `f3d9c38` | `chore:` | Enable DB transactions and garbage collection by default |
| Dec 19 | `8f5f70c` | `docs:` | Add Tusky migration guide |
| Dec 19 | `80c7954` | `docs:` | Update image reference for quilts |
| Dec 29 | `651aea8` | `docs:` | Revisions to operations subsection |
| Dec 29 | `68c9ba2` | `docs:` | Style guide revisions to walrus-sites docs |
| Dec 30 | `71dd1da` | `chore:` | Improve error reporting in `WalrusReadClient` |

#### Activity breakdown by commit type

| Date | fix | feat | chore | docs | test | ci | Total | Signal |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| Dec 2 | 1 | 1 | 1 | 0 | 1 | 1 | 5 | Substantive node work |
| Dec 4 | 0 | 0 | 1 | 0 | 0 | 0 | 1 | Substantive node work |
| Dec 5 | 1 | 1 | 0 | 0 | 0 | 0 | 2 | Substantive node work |
| Dec 6–7 | — | — | — | — | — | — | 0 | **Gap** |
| Dec 8 | 1 | 1 | 0 | 0 | 0 | 0 | 2 | Substantive node work |
| Dec 9 | 0 | 0 | 1 | 1 | 0 | 0 | 2 | Covert removal under `chore:` |
| Dec 10 | 3 | 0 | 2 | 1 | 0 | 0 | 6 | Three urgent fixes in one day |
| Dec 11 | 0 | 2 | 2 | 5 | 0 | 2 | 11 | Docs flood begins |
| Dec 12 | 1 | 0 | 0 | 1 | 0 | 0 | 2 | |
| Dec 13 | 0 | 0 | 0 | 0 | 1 | 0 | 1 | Test suppression commit |
| Dec 14 | — | — | — | — | — | — | 0 | **Gap** |
| Dec 15–19 | 0 | 1 | 2 | 16 | 0 | 1 | 20 | Documentation noise only |
| Dec 20–28 | — | — | — | — | — | — | 0 | **9-day silence** |
| Dec 29–30 | 0 | 0 | 1 | 2 | 0 | 0 | 3 | Docs + minor chore |

After Dec 13, every substantive fix has been merged. The repository then enters a 9-day hard silence (Dec 20–28), followed by documentation-only commits. There is no further node-level `fix:` commit for the remainder of the month. The stabilization window is Dec 2–13: eleven days, all critical changes, all under suppressive commit labels.

#### A. Epoch desync — the 50ms-to-30s proof (Dec 9, `c6e920a`)

This commit removes the entire `SuiSystemEventProvider` legacy polling fallback and locks nodes into the checkpoint-based event processor. The forensic tell is the test change:

```diff
-    tokio::time::sleep(Duration::from_millis(50)).await;
-    check_that_blob_is_not_available(client, blob_id).await
+    // Wait for the deletion event to be processed by storage nodes.
+    // The checkpoint-based event processor needs time to process the deletion.
+    wait_for_blob_to_be_unavailable(client, blob_id, Duration::from_secs(30)).await
```

The 50ms sleep encoded the assumption that deletion events propagated to nodes within 50ms — valid under the legacy `SuiSystemEventProvider` which polled Sui events directly and in process. The checkpoint-based processor introduces a pipeline lag of up to 30 seconds between when an event is emitted on-chain and when a storage node can act on it. During that window, nodes serve stale state: a blob that has been deleted on-chain is still served as available by any node that has not yet processed the deletion checkpoint. This is the epoch desync. It is not a documentation issue; it is a live data-consistency window that existed on mainnet while the legacy provider code was being used in production.

The commit also removes `use_legacy_event_provider` from `node_config_example.yaml`, `deploy.rs`, `runtime.rs`, and `node.rs`, eliminating the operator-facing fallback without any public announcement:

```diff
-    system_events::{EventManager, SuiSystemEventProvider},
+    system_events::EventManager,

-    pub use_legacy_event_provider: bool,
```

#### B. Race condition and deadlock at epoch boundary (Dec 2, `6aba4f7`)

Fixed a deadlock in `BlobRetirementNotifier::drop()`. When a blob expired or was deleted at an epoch boundary, the `Drop` implementation attempted to acquire the same mutex already held by the notification path, causing the storage node thread to hang indefinitely. The fix introduced a `cleanup_blob_retirement_notify` method with explicit ref-count checking and added metrics tracking:

```diff
-    pub fn acquire_blob_retirement_notify(&self, blob_id: &BlobId) -> BlobRetirementNotify {
+    fn acquire_blob_retirement_notify(&self, blob_id: &BlobId) -> BlobRetirementNotify {
```

The method was also demoted from `pub` to private, preventing external callers from triggering the deadlock path directly. A node that deadlocks on blob retirement stops serving all shard requests for that blob until it is restarted — exactly the symptom of an epoch desync as seen by external observers.

#### C. Removal of `SequentialProcessor` (Dec 4, `a0fe265`)

The `BlobEventProcessorImpl` enum previously had two variants: `BackgroundProcessors` (parallel workers) and `SequentialProcessor` (fallback for zero workers, preserving legacy serial behavior). The `SequentialProcessor` branch was the safe fallback that guaranteed ordering of blob events per blob ID. This commit deleted it entirely:

```diff
-enum BlobEventProcessorImpl {
-    BackgroundProcessors { ... },
-    SequentialProcessor {
-        background_event_processor: Arc<BackgroundEventProcessor>,
-    },
+pub struct BlobEventProcessor {
+    node: Arc<StorageNodeInner>,
+    background_processor_senders: Vec<UnboundedSender<TrackedBackgroundTask>>,
+    ...
 }
```

Simultaneously, `num_workers: usize` was changed to `NonZeroUsize`, making it impossible to configure zero workers and therefore impossible to reach the now-deleted sequential path. Any operator running the old binary with `num_workers: 0` in config would previously have gotten deterministic serial processing. After this commit they get a startup error instead.

#### D. Three urgent fixes in one day (Dec 10)

Three separate `fix:` commits landed on Dec 10 — the highest single-day fix count in the diff — all targeting node infrastructure:

- `5eb4ba4` — **WAL-874 and WAL-876: shutdown coordination.** The `BlobEventProcessor` was not owned by `StorageNode` at construction time; it was created ad hoc. This commit restructures ownership so the processor is tracked by the node, enabling clean shutdown. The internal Jira tickets WAL-874 and WAL-876 are referenced directly in the commit message, confirming this was a known tracked defect.
- `236d0ea` — **Recovery metric.** Added a counter to track how often recovery falls back to non-deferred mode, making the frequency of this edge case visible for the first time.
- `c9af789` — **Race between catchup and checkpoint tailing.** The `EventBlobCatchupManager` could enter `process_event_blob` while the checkpoint tailing goroutine was still running, causing both to write to shared checkpoint state concurrently. The fix adds an explicit invariant check:

```diff
+        if !coordination_state.is_tailing_stopped.load(Ordering::Acquire) {
+            return Err(CatchupError::Recoverable(anyhow::anyhow!(
+                "checkpoint tailing should be stopped when process_event_blob is running"
+            )));
+        }
```

This race is another source of the epoch desync: during a catchup pass, the node's view of which epoch it is in can diverge from the live chain state if tailing is running concurrently.

#### E. Test suppression (Dec 13, `0173958`)

Labeled `test: various test improvements`. Actual changes:

1. Deleted the 113-line `correctly_handles_blob_deletions_with_concurrent_instances` test from `node.rs`. This test covered the exact failure mode exposed by the race condition fixed on Dec 2 and Dec 10 — concurrent instances processing deletions for the same blob at epoch boundaries.

2. Added `#[ignore = "ignore long-running test by default"]` to:
   - `recovers_all_shards_for_multi_shard_node` — tests multi-shard recovery across an epoch change
   - `deletes_expired_blob_data` — tests that blobs with past end-epoch are actually deleted from storage

3. Moved `MarkMetadataStored(true)` and `MarkMetadataStored(false)` from the `#[should_panic]` group to the `expected_success` group in `blob_info.rs`:

```diff
-            #[should_panic] metadata_true: (BlobInfoMergeOperand::MarkMetadataStored(true)),
-            #[should_panic] metadata_false: (BlobInfoMergeOperand::MarkMetadataStored(false)),
 ...
+            metadata_true: (BlobInfoMergeOperand::MarkMetadataStored(true)),
+            metadata_false: (BlobInfoMergeOperand::MarkMetadataStored(false)),
```

Storing metadata on a `New` blob state was previously expected to panic — the operation was semantically invalid. After this change it is expected to succeed. This is a behavior change, not a test cleanup.

4. Reduced test batch sizes from 1000 to 10 in `GarbageCollectionConfig::default_for_test()`, reducing the amount of data processed per test run and making it less likely for tests to exercise the full GC pipeline at scale.

The deleted test was re-implemented as a simtest in `walrus-simtest/tests/simtest.rs` — but only under `#[ignore = "ignore integration simtests by default"]`, meaning it does not run in CI by default and provides no regression protection.

#### F. Config flip under `chore:` (Dec 19, `f3d9c38`)

Enabled garbage collection and data deletion in production defaults:

```diff
-  enable_blob_info_cleanup: false
-  enable_data_deletion: false
+  enable_blob_info_cleanup: true
+  enable_data_deletion: true

-    experimental_use_optimistic_transaction_db: false
+    use_optimistic_transaction_db: true
```

The `experimental_` prefix removal is significant. This flag was introduced as a temporary opt-in for operators willing to test optimistic transaction semantics. Removing the prefix and flipping the default to `true` in a `chore:` commit silently pushes every operator who upgrades to a new transaction mode without any migration guide, changelog entry, or advisory. Operators who set `experimental_use_optimistic_transaction_db: false` explicitly in their config files will have their setting silently aliased to the new key via `#[serde(alias = "use_optimistic_transaction_db")]` — but only on the read path; writes will use the new key name.

The commit message references "test results on PTN, Testnet, and Mainnet," confirming these settings had been tested on live infrastructure before the default was flipped.

---

## Support This Work

The DeFi ecosystem relies on independent whitehats who fight uphill battles against heavily funded protocols and the bug bounty platforms — Cantina, HackenProof — that actively collude with them.

If my research has protected your liquidity or you support independent security accountability, consider funding my infrastructure at my [GitHub Sponsors page](https://github.com/sponsors/donnyoregon).

---

> `DECEMBER_CODE_CHANGES.diff` is derived from [MystenLabs/walrus](https://github.com/MystenLabs/walrus), which is licensed Apache 2.0. Reproduced here under the same license for public interest documentation.
