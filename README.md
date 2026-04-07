# GD_EYE Modular Anti-Cheat

GD_EYE is a modular Roblox anti-cheat system built around server authority, secure networking, client hardening, and flexible enforcement.

## Architecture

- `Shared` (`ReplicatedStorage`): protocol, crypto, validation, and central config.
- `Server` (`ServerScriptService`): authoritative validation, scoring, enforcement, logging, and ban storage.
- `Client` (`StarterPlayerScripts`): tamper/hook/sandbox/executor detections and silent flag reporting.

## Core Principles

- Never trust the client.
- The server validates physical behavior, remote payloads, and state transitions.
- Multiple signals are required before hard enforcement (lower false-ban risk).
- Critical anti-ban signals can be persisted immediately.

## Secure Networking

- Dynamic secure channel name per server boot (16-char runtime remote).
- Bootstrap channel provides secure remote name only at runtime.
- Handshake challenge/response using `Crypto.Sign(...)`.
- One-time salts on handshake/canary challenges (replay hardening).
- Canary challenge loop with trust model:
  - initial canary after handshake,
  - periodic re-checks,
  - trust revocation after repeated misses/timeouts,
  - multi-family proof rotation (`A/B/C`),
  - toggle family rotation via `Hardening.CanaryFamilyRotationEnabled`,
  - support-friendly mode via `Hardening.CanaryStrictEnforcement = false`.
- Signed payload envelopes for remote traffic:
  - `seq` (anti-replay),
  - `ts` (clock skew check),
  - `sig` (tamper detection),
  - `body` (payload).
- Server-issued action tokens:
  - `state` token for heartbeat/state checks,
  - `action` token for client-flag/high-risk action flows.
- Remote schema + payload shape validation:
  - type checks,
  - string length limits,
  - table size limits,
  - nesting depth limits.
- Strict rate limiting:
  - per-second window,
  - 5-second burst limit.
- Remote hygiene tiering:
  - critical actions can use stricter per-second caps,
  - optional mode to require signed envelopes only for critical traffic.

## Honeypots and Decoys

- Honeypot remotes in `ReplicatedStorage` (fake remotes).
- Decoy actions (fake exploit targets):
  - `CLAIM_ADMIN_REWARD`, `GET_FREE_CURRENCY`, `SYNC_SERVER_CASH`.
- Decoy protocol actions:
  - `UPDATE_CASH`, `TELEPORT_REQUEST`, `SYNC_INVENTORY_STATE`.
- Honeypot/decoy triggers produce high-confidence enforcement signals.

## Client Hardening

- Main script stealth:
  - runtime rename to benign aliases,
  - runtime reparent to less obvious `PlayerScripts` locations,
  - optional startup delay to reduce static startup fingerprints,
  - optional runtime alias rotation,
  - optional parent shuffle across `PlayerScripts` folders,
  - optional decoy folder creation for additional noise,
  - optional randomized scanner order for polymorphic behavior.
- Noise routines for analysis misdirection (`GetLocalThreadIdentity`, `ValidateNetworkEntropy`).
- Integrity:
  - script existence checks,
  - function DNA checks (`FireServer`, `task.wait`, `task.spawn`).
- AntiHook:
  - `__namecall`, `__index`, `__newindex` integrity checks,
  - metatable poisoning probes.
- AntiDebug:
  - timing stall detection,
  - heartbeat attestation drift,
  - thread suspend indicators.
- EnvironmentScan:
  - executor global scans (`getgc`, `hookfunction`, `hookmetamethod`, etc.),
  - executor/anti-ban signatures (`potassium`, `rflhub`, `antikick`, etc.),
  - honeypot probes in `_G`/`shared`.
- AntiSandbox:
  - runtime primitive tamper checks,
  - environment/metatable anomaly checks.
- CoreGui scanning:
  - baseline capture,
  - injected GUI/script descendant detection,
  - auto-learning suspicious GUI signatures.
- Watchdog self-healing monitor:
  - periodic module existence checks,
  - emits `Tamper_0x99_*` on module deletion/tamper,
  - stall pulse guard emits `Tamper_0x9A_watchdog_stall` on scanner freeze/suspend.

## Server Detection Engine

- Movement:
  - speedhack detection with latency buffer,
  - 1-second distance-window validation,
  - teleport burst detection,
  - leaky-bucket anomaly accumulation,
  - time-window score bonus for repeated suspicious movement,
  - server prediction + rollback marker for impossible displacement spikes.
- Noclip:
  - periodic checks + raycast-based signals.
- Character/state sanity:
  - anomalies in root part, assembly, and humanoid state.
- Replication abuse checks.
- Tool abuse checks.
- Optional football checks:
  - active only when `Config.Game.Football = true`
  - and `Config.Features.FootballLegacyChecks = true`.

## Server Authority Helpers

- `ServerAuthority` includes server-side validation helpers such as:
  - action-vs-state sanity (dead/seated/menu),
  - noclip ray helper,
  - recalculation/clamping of sensitive values,
  - action-specific payload validators (`SHOOT_REQUEST`, `BUY_REQUEST`, `TELEPORT_INTENT`, `REWARD_CLAIM`).

## Offender Memory and Signature Store

- `SignatureStore` stores:
  - learned signatures,
  - offender history by `UserId`.
- Repeat offenders can receive score boosts on join.
- Repeated low-level flags can accumulate suspicion over time.

## Enforcement Model

- Modes:
  - `ScoreBased`,
  - `InstantKick`,
  - `InstantBan`.
- Per-reason actions:
  - `ban`, `kick`, `warn`, `scorebased`, `silentflag`.
- Required hits + hit decay.
- Quorum logic:
  - requires multiple signals,
  - requires distinct signal families,
  - requires score floor before trigger.
- Delayed ban queue + manual ban-wave support.

## Logging and Webhooks

- Discord webhook support with separate channels:
  - flags, stats, bans, kicks, warnings, ban waves.
- Event embeds include player, reason, score, and action.
- `LogOnlyMode` can be used for safe calibration/testing.

## Admin Commands (Ban Wave)

- `;gdeye queueban <userId> <reason>`
- `;gdeye wave list`
- `;gdeye wave execute`
- `;gdeye wave clear`
- `;gdeye unban <userId>`
- `;gdeye lookup <userId>`

## Tuning (False Bans vs. Aggressiveness)

- Reduce false positives:
  - increase `ExecutionQuorum.MinScoreForTrigger`,
  - increase `ExecutionQuorum.RequiredSignals`,
  - increase `CanaryResponseTimeout`,
  - raise `KickScore`/`BanScore`.
- More aggressive anti-bypass:
  - lower `RequiredHits.Default`,
  - lower `CanaryMaxMissesBeforeTrustDrop`,
  - expand `AntiBanCriticalPatterns`,
  - keep `ImmediatePersistCriticalBans = true`.
- Client stealth tuning:
  - `Config.ClientStealth.Enabled`
  - `Config.ClientStealth.RuntimeAliasRotate`
  - `Config.ClientStealth.ParentShuffleEnabled`
  - `Config.ClientStealth.RandomizeScannerOrder`
  - start conservative and measure client FPS/latency before increasing aggressiveness.

## Decoy Config Data

- `Config.Network_Optimizations` is an intentional decoy table for misdirection.
- Authoritative checks instead use server thresholds and server-side validation.

## Installation

See `INSTALL.md` for hierarchy and step-by-step setup.
