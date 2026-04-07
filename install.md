# GD EYE Modular Install

For full feature documentation and architecture details, see `README.md`.

Create this hierarchy in Roblox Studio:

- `ReplicatedStorage`
  - `GD_EYE_Shared` (Folder)
    - `Config` (ModuleScript)
    - `Crypto` (ModuleScript)
    - `Validation` (ModuleScript)
    - `Protocol` (ModuleScript)
- `ServerScriptService`
  - `GD_EYE_Server` (Folder)
    - `Main` (Script)
    - `Controller` (ModuleScript)
    - `RemoteSecurity` (ModuleScript)
    - `BanStore` (ModuleScript)
    - `SignatureStore` (ModuleScript)
    - `Detections` (Folder)
      - `Movement` (ModuleScript)
      - `Heartbeat` (ModuleScript)
      - `Replication` (ModuleScript)
      - `ServerAuthority` (ModuleScript)
      - `FootballLegacy` (ModuleScript)
      - `Noclip` (ModuleScript)
      - `CharacterTamper` (ModuleScript)
      - `ToolAbuse` (ModuleScript)
      - `StateSanity` (ModuleScript)
      - `BallOwnership` (ModuleScript)
      - `CharacterSanity` (ModuleScript)
      - `StateAbuse` (ModuleScript)
- `StarterPlayer`
  - `StarterPlayerScripts`
    - `GD_EYE_Client` (Folder)
      - `Main` (LocalScript)
      - `Controller` (ModuleScript)
      - `Detections` (Folder)
        - `EnvironmentScan` (ModuleScript)
        - `AntiHook` (ModuleScript)
        - `AntiDebug` (ModuleScript)
        - `Integrity` (ModuleScript)
        - `CoreGui` (ModuleScript)
        - `MobileExecutor` (ModuleScript)
        - `AntiSandbox` (ModuleScript)
        - `FunctionIntegrity` (ModuleScript)
        - `SchedulerHealth` (ModuleScript)
        - `MetamethodMatrix` (ModuleScript)
        - `RuntimeAttestation` (ModuleScript)
        - `DecoyPaths` (ModuleScript)
        - `Watchdog` (ModuleScript)

## Important

- Keep webhook URLs only in server config.
- Enable HTTP Requests if Discord webhook logging is enabled.
- Tune thresholds in `Config` before production rollout.
- `GulleUILibrary` is treated as a cheat signature (blacklist), not whitelisted.
- Football checks master toggle:
  - set `Config.Game.Football = true/false`
  - legacy football protections are also gated by `Features.FootballLegacyChecks`
  - if either toggle is off, football-specific checks are disabled
  - includes reach/ownership distance validation, stamina abuse checks, ball and shot-power anomaly checks.
- Advanced client hardening includes:
  - anti-sandbox primitive checks
  - mobile-executor signatures
  - stronger metamethod hook detection (`__namecall`, `__index`, `__newindex`)
  - metamethod matrix + rotating probes
  - scheduler health drift/gap checks
  - runtime attestation challenges
  - decoy token/path tamper checks
  - client-side signal quorum correlation
  - runtime stealth profile (`Config.ClientStealth`) with alias rotation, parent shuffle, and scanner-order randomization.
- Server anti-bypass hardening includes:
  - handshake gating (actions blocked before valid handshake)
  - handshake timeout + re-challenge
  - periodic canary re-check with trust revocation after repeated misses
  - honeypot remotes (firing them triggers high-confidence enforcement)
  - decoy actions (fake exploit-target actions that instantly trigger enforcement)
  - signed secure payload envelopes for heartbeats/flags/canary responses (anti-replay + anti-tamper)
  - canary family rotation (`A/B/C`) with toggle support
  - server-issued `state` + `action` tokens for client traffic validation
  - remote hygiene tiering for critical action traffic
  - action-specific server validators for `SHOOT_REQUEST`, `BUY_REQUEST`, `TELEPORT_INTENT`, `REWARD_CLAIM`
  - movement leaky-bucket scoring + time-window accumulation
  - server-side rollback marker for impossible movement spikes.
  - global server physics engine with anti-tamper correction.
  - server-side webhook field sanitization (anti-mention/payload-abuse hardening).
  - remote limiter/handshake state cleanup when players leave.
  - atomic signature/offender datastore updates (`UpdateAsync` based).
- Auto-learning signatures:
  - suspicious CoreGui names are stored in DataStore
  - repeated signatures are auto-promoted for future enforcement.
- Message customization:
  - edit `Config.Messages.Kick`, `Config.Messages.Ban`, and `Config.Messages.BanJoin`.
- Executor/anti-ban signature coverage:
  - detects `hookmetamethod` usage
  - detects runtime executor tokens (`potassium`, `rflhub`, `gulleui`, etc.)
  - detects anti-ban token patterns (`antikick`, `anti_ban`, `kickbypass`, etc.).

## High-Robustness Tuning

- In `Config.Network`, tune payload and burst guards:
  - `BurstLimitPer5Seconds`
  - `MaxStringPayloadLength`
  - `MaxTableEntries`
  - `MaxNestingDepth`
- In `Config.Performance`, tune server load behavior:
  - `PlayerChecksPerTick`
  - `HeavyCheckEveryTicks`
  - `BallCheckEveryTicks`
  - `ControllerYieldEveryPlayers`
- In `Config.AdvancedProfiles`, choose a preset and align thresholds for your game mode.
- Canary/support toggles in `Config.Hardening`:
  - `CanaryFamilyRotationEnabled` (`false` = single-family canary)
  - `CanaryStrictEnforcement` (`false` = softer canary enforcement, better for support-ticket review flow)
- Remote/action/movement additions:
  - `Config.RemoteHygiene` (critical traffic rules)
  - `Config.ActionValidators` (payload constraints per high-risk action)
  - `Config.MovementScoring` (bucket + window tuning)
  - `Config.Rollback` (rollback trigger/cooldown tuning)
  - `Config.ClientStealth` (startup jitter, alias/parent polymorphism, decoy folders).
  - `Config.ClientAdvanced` (client signal quorum + scheduler health tuning).
  - `Config.PhysicsEngine` (global physics simulation, material friction/restitution, shot handling).
  - `Config.Logging` webhook channels (safe telemetry routing; keep URLs private).

## Physics Engine Setup

- The server physics engine is global and not football-only.
- Tracked parts are selected by:
  - name match in `Config.PhysicsEngine.TargetPartNames`, or
  - attribute `TrackPhysicsEngine = true`, or
  - attribute named by `Config.PhysicsEngine.TrackAllUnanchoredWithAttribute` (default `PhysicsSim`) set to `true`.
- For custom projectiles/balls, set one of those attributes on the part.
- Material-specific behavior:
  - tune `Config.PhysicsEngine.MaterialFriction`
  - tune `Config.PhysicsEngine.MaterialRestitution`
  - keys use material enum strings (for example `Enum.Material.Ice`, `Enum.Material.Grass`, `Enum.Material.Concrete`).
- Knuckle shot:
  - send `SHOOT_REQUEST` with `shotType = "knuckle"` and `power`.
  - optional `lift` controls vertical component.

## Manual Ban Wave Commands

Admin commands are enabled through `Config.Admin`:

- Add your user ID to `Config.Admin.AdminUserIds`
- Set command prefix in `Config.Admin.CommandPrefix` (default `;gdeye`)

Commands:

- `;gdeye queueban <userId> <reason>`
- `;gdeye wave list`
- `;gdeye wave execute`
- `;gdeye wave clear`
- `;gdeye unban <userId>`
- `;gdeye lookup <userId>`

Webhook:

- Set `Config.Logging.Webhooks.BanWaves` for queue/execute/clear/unban/lookup logs.
