# Hardware Agents and Board Deployment

## Scope and boundaries

Use this profile when building an agent that runs on a microcontroller or installing it on a physical board. It owns device-specific installation, resource, power-loss, and commissioning contracts, not a new model, training method, or autonomy level. The supporting implementation evidence and incident lineage are in [source links](source-links.md#hardware-agent-session-mining).

First distinguish where inference happens: on the board, at a remote model service, or in a host process. A board can own the agent loop, credentials, tools, memory, and wake schedule while inference remains remote. “Flashed” does not imply on-device inference or independence from a laptop.

Start with one read-only model/tool cycle on the chosen device. Recurring autonomy, programmable tools, broad discovery, and physical actuation are post-MVP unless explicitly requested. Do not deploy to a connected board merely because the user asked for a design or session analysis.

| Shared concern | Canonical owner |
|---|---|
| basic loop, retries, and budgets | [agentic loop](agentic-loop.md) |
| typed calls, permission decisions, receipts, and resulting-state limits | [tools and permissions](tools-and-permissions.md) |
| durable working memory and compaction | [context, memory, and compaction](context-memory-compaction.md) |
| discovery, schema binding, and drift | [environment-adaptive tools](environment-adaptive-tools.md), [skills and connectors](skills-and-connectors.md) |
| scheduled goals and continual runtimes | [planning and goals](planning-and-goals.md), [recursive and continual harnesses](self-refining-recursive-harnesses.md) |
| credentials, injection boundaries, trace handling, and incidents | [security and observability](security-observability.md) |
| public external communication | [public-board communication](skills-and-connectors.md#agent-communication-via-public-boards) |
| failure probes and launch methodology | [hardware evals](evals.md#hardware-agent-evals) |

## Establish the target before choosing an installer

Record the exact board/revision, chip, flash size, available external RAM, firmware/interpreter version, peripheral wiring, power source, USB identity, and current boot path. A generic development-board build label is not proof of the physical board's memory mode or display pin map. Keep existing board-support initialization until a replacement has been verified against the board revision.

Inventory resource headroom rather than assuming that advertised flash is RAM. Measure free and largest allocatable blocks in both interpreter/application memory and the native networking heap where available. Include framebuffer, task stacks, TLS handshakes, certificate chains, request strings, response parsing, and storage buffers in peak estimates.

Inspect the installed bootloader, partition table, application selection, filesystem, and any launcher before proposing a write. Prefer non-disruptive inspection. Entering an interpreter's raw REPL usually interrupts the running app; opening a serial monitor may toggle reset lines. Disclose those effects, use the least disruptive supported route, and restore normal boot after an approved inspection.

| Installation route | Required evidence | Write boundary |
|---|---|---|
| compiled application update | compatible image, actual partitions, selected app slot, image fit, persistent-state compatibility | only the verified application range; preserve bootloader, partition table, selection metadata, settings, and filesystem unless separately included in the approved change |
| interpreter/launcher app | actual app discovery/import convention, helper paths, startup/exit behavior, available filesystem space | add app/modules through the existing installer or file-transfer interface; preserve other apps and the launcher |
| base firmware replacement | explicit need for a runtime/boot layout change, full recovery image, migration/reset decision | separately approved firmware and data ranges; never infer permission to erase identity or credentials |

A menu row may be discovered by importing a filename, not by flashing a binary or installing a manifest. Sample two existing apps and read the actual launcher's code. Check whether module import must call an entry point, how errors return to the menu, and whether restart is the exit mechanism. Autostart is a separate, reversible change with a documented bypass to recover the menu or REPL. Avoid loading the entire menu/UI before autostart if that consumes the app's scarce heap.

Do not guess an application offset, RAM configuration, reset-key sequence, or device path from a similar board. USB paths can change after a reset; rediscover by stable device identity before another operation.

## Trusted runtime and typed state

The device runtime, not the model, owns boot/storage validation, credential access, transport limits, dispatch, action recording, sleep deadlines, and recovery. Keep model-directed tools narrower than firmware maintenance capabilities: permission to use a calculator or scheduler is not permission to rewrite flash, change networking, read secrets, or restart hardware.

Use versioned records appropriate to the storage available:

| Record | Device-specific contract |
|---|---|
| deployment manifest | board/runtime compatibility, source and artifact hashes, build target, write ranges, state schema, migration, rollback, and evidence paths; no credentials |
| identity and working checkpoint | bounded persona/task memory, context references, exact cursor types, schema version, and recoverable old version |
| credential state | separately protected configuration, refresh ownership, durable token rotation, and redaction; assume a physical flash backup can contain credentials |
| outbound action state | pending intent/action key before dispatch; receipt or unknown outcome afterward; survive reset before and after remote acceptance |
| wake state | requested wake, runtime retry floor, server rate-limit floor, fired-job identity, and sleep reason; distinguish these fields |
| recovery state | boot/reset reason, failure domain/count, firmware version, storage health, and last completed checkpoint |

For a state update, write and validate a candidate before replacing the known-good version. Confirm the filesystem's rename and power-loss guarantees; a temporary-file scheme that deletes the old record before rename is not automatically atomic. Keep a recoverable previous copy or journal when those guarantees are insufficient. Bound event logs and avoid rewriting unchanged state every idle tick to limit flash wear.

Use the ordinary tool-result contract with a small device error vocabulary, for example `body_too_large`, `parse_depth`, `stream_incomplete`, `tls_clock_invalid`, `memory_low`, `storage_unavailable`, `outcome_unknown`, and `approval_required`. Distinguish retryable read/transport failures from irreversible or uncertain action outcomes. On storage corruption or failed checkpointing, pause writes instead of silently creating a new identity or formatting an established filesystem.

## Fit transport and parsing to the board

Set independent bounds for request construction, raw wire bytes, decoded content, JSON string length/depth, context history, cached observations, and tool pages. Library defaults may be smaller than the configured context: inspect integer widths and allocator behavior in the exact dependency revision. Test serialization as well as deserialization. Increasing a network body cap alone will not fix a parser depth or string-width failure.

For large responses, process bounded chunks and keep only fields the loop needs. A streamed model answer can be much smaller than its raw event stream. Do not buffer all reasoning/metadata events just to retain the answer, and do not expose private reasoning in diagnostics. Bound event-line size, cumulative wire bytes, retained answer size, and malformed-event handling independently.

Handle content length, chunked encoding, and connection-close framing explicitly. For event streams, support transport fragments that split lines, JSON, or UTF-8 characters; retain completion/usage signals needed by the adapter. A disconnected partial answer is not a valid action. Dispatch only after the complete envelope passes the same local schema and permission checks as non-streamed output.

Separate connection/TLS, first-response/header, idle-body, and total-transfer deadlines. Verify units and storage widths: a nominal long timeout may silently wrap in a narrower library API. Streaming permits progress over a longer bounded transfer, not unlimited waiting. Feed watchdogs on measured progress without defeating the total deadline; ensure blocking header/TLS waits fit the task watchdog budget.

Close sockets and release payloads on every success, exception, timeout, and cancellation path. Test repeated TLS handshakes and token refresh while context grows. Interpreter free memory alone may miss fragmentation or exhaustion in the native TLS heap. A low-memory restart is a last-resort recovery at a recorded safe boundary: reconcile pending effects, checkpoint the remaining sleep, and resume it rather than starting an extra cycle. Do not reboot-loop on an unfixable allocation problem.

Byte limits are not character limits. Exercise UTF-8 network names, multi-byte model text, and escaped Unicode surrogate pairs on the actual interpreter. Convert surrogate pairs before parsing/persisting when the pinned runtime requires it. Clip cosmetic screen fields at valid character boundaries without discarding an otherwise valid action; never clip identifiers, arguments, approval content, or memory silently. If recovering JSON from surrounding narration is supported, require one unambiguous schema-valid envelope and a bounded corrective retry with field-specific errors.

## Connectivity, clock, and wake behavior

Keep network credentials outside public source/releases. Multiple known networks and hidden-network fallback are optional product requirements, not universal defaults. Test the actual radio/runtime: encoding a network name correctly does not prove that it joins. Reconnection should be bounded and retain checkpointed work rather than rerunning initialization.

Probe certificate verification, hostname checking, trust roots, TLS API shape, and the runtime's epoch on the real device. Port compatibility with desktop Python does not prove embedded TLS compatibility. Keep certificate/hostname verification enabled. Prefer a fixed runtime over a clock hack; any necessary compatibility adapter must be confined to a pinned defect, preserve logical time, and survive background time synchronization without weakening verification.

Document how time is established before certificate validation. A Date header received only after verified HTTPS may not solve a cold-boot invalid clock; unauthenticated network time is a trust assumption, not authenticated evidence. Clock bootstrap and later trusted correction need an explicit policy. Never generalize an epoch shift or plain-HTTP fallback from one build to another.

Use monotonic time for durations and a defined UTC clock for durable calendar jobs. Specify what happens on clock correction, missing time, restart, overdue jobs, and multiple jobs becoming due together. A cron-like scheduling tool may enqueue instructions for a later agent cycle; it is not necessarily an operating-system shell. Do not execute job text as firmware commands merely because it uses familiar syntax.

Define cadence as start-to-start or sleep-after-cycle; account for cycle duration and declare missed-wake/coalescing behavior. When recurring operation is requested, enforce the configured sleep ceiling through every route: model sleep, scheduler, retry ladder, saved wakes at boot, and migration. Server rate limits remain a minimum wait even when longer than the normal cadence. Do not clear that floor when resetting version-specific failures. Keep authentication, parsing, transport, storage, and resource failures distinguishable so one successful read does not erase a persistent fault in another domain.

A long backoff can make a healthy device appear silent. Report the reason and absolute next wake. Define which successful work resets a retry ladder; requiring a perfect whole cycle can pin it at maximum after one slow model call. Compaction failure can preserve the old checkpoint and defer while context remains within a hard ceiling; once the ceiling is reached, pause safely rather than growing forever. Preserve the user's intended durable goals instead of accumulating accidental retry notices into permanent behavioral prohibitions.

## State-preserving installation and rollback

1. Identify the target and approved installation route. Stop or hand off the sole serial reader; do not kill unrelated processes. Capture current firmware, identity/checkpoint metadata, wake state, and recovery access without publishing secrets.
2. Pin dependencies and build configuration. Verify that the artifact is the physical-device target, not an emulator, simulator, or host-bridge build. Package the exact artifact with its source revision/hash, checksum, expected size, and deployment manifest.
3. For compiled updates, compare device and package partitions, confirm the boot-selected slot and image fit, and preserve a verified recoverable backup appropriate to the changed ranges and valuable state. Initial/unknown layouts may need a full flash backup; repeat compatible app-only updates need not always repeat it. Back up filesystem apps and the launcher for interpreter updates. Keep backups private.
4. Check boot-time migrations, mount/format behavior, and persistent selection/settings flags. An app-only write can still lose identity if startup auto-formats on mount failure. Established storage must fail closed; formatting or resetting it needs an explicit first-provisioning/reset decision.
5. Perform only the approved write or file transfer. Verify its hash/read-back, restore ordinary boot, and rediscover the device identity if the port changes. File installation must also prove that the correct app appears or autostarts without replacing other apps.
6. Verify version, retained identity, checkpoint/migration result, applicable network/clock paths, model/tool cycle, any requested sleep/wake behavior, and remote read-back before claiming deployment health. Preserve earlier evidence when a subsequent flash interrupts a soak; the new image starts a new observation window.
7. Document a tested rollback route. Older firmware must understand the current state schema, or rollback needs a compatible state backup/migration. Restore only the required ranges; never blindly replay an old full flash image over newer credentials, receipts, or server rate-limit state.

Do not supply a copy-paste destructive flash command with guessed values. An actionable deployment handoff names the measured chip/port, installer/version, artifact/hash, ranges, preserved state, and recovery route. Account setup and public posting are separate actions governed by available capabilities and actual authorization; neither invent a blanket prohibition nor infer those permissions from permission to install firmware.

## Commissioning and evidence

Keep evidence levels separate:

| Level | What it proves | What it does not prove |
|---|---|---|
| native fixture/parser checks | local bounds, framing, state, and envelope behavior | physical networking, allocator, watchdog, or peripherals |
| component emulator | behavior of the components actually exercised | that the production agent loop ran |
| production-loop emulator with host transport | control flow, selected recovery paths, separately authorized remote effects | device TLS, streaming if the bridge uses non-streamed responses, radio, display, or real-time uptime |
| artifact write verification | intended bytes reached the selected target | successful boot, retained identity, or useful work |
| physical commissioning | boot and observed cycle/sleep/wake paths on this image | long-duration reliability beyond the observed window |

For emulation, declare clock scaling and active versus wall time, host-sleep behavior, transport substitutions, and reset/stop conditions. Use separate test credentials/state for authorized external effects; a replay can otherwise duplicate production activity or compete for token refresh. Never upload the emulator target to the physical board.

One owner holds the serial port at a time. Check that ownership, device enumeration, reset reason, and heartbeat before diagnosing a reboot from missing console output. Observation should avoid reset/REPL transitions where supported. A real native-USB wedge may require a physical replug; report that need instead of entering a repeated-flash recovery loop.

Expose phase start/elapsed time, last completed action, sleep reason/deadline, reset reason, memory/largest-block telemetry, parser/transport counters, and checkpoint result. Keep display status separate from durable message text; a stale “remembering” label during sleep is not evidence of an active compaction. Poll buttons between blocking operations when possible, discard unintended queued wake presses, and retain a hardware recovery path.

Do not call the screen or keyboard verified from console logs alone. Inspect the physical rendering/input or explicitly hand that check to the operator. Likewise, a transport success or model claim is not a remote action receipt: retain server IDs and read back resulting state. Typed inbox cursors and post IDs are different identifiers; acknowledging the wrong one can leave already handled work unread.

When safe reset/power-cycle recovery is part of the deployment, its tests must show retained identity, safely reconciled effects, refreshed credentials when due, and any expected remaining sleep/wake. State the exact observation duration and remaining gaps. A clean first cycle or hour does not close an overnight freeze investigation. Use the focused probes in [evals](evals.md#hardware-agent-evals) and the [hardware checklist](checklists.md#hardware-agent-and-board-deployment-checklist) rather than another evaluation framework.

## Build sequence and anti-patterns

Build board inventory and a recovery path first, then a read-only boot/network/verified-TLS probe, one bounded remote-model call, one typed read tool, durable bounded memory when needed, state-preserving installation, and physical commissioning. Exercise sleep/wake when recurring operation is requested. Add writes only under the existing approval policy and after reset/unknown-outcome recovery works. Add scheduling, larger catalogues, programmable computation, UI polish, and unattended operation only when the requested product and measured results justify them.

Avoid these shortcuts:

- replacing base firmware when adding a filesystem app would suffice;
- treating more flash, external RAM, or streaming as an unlimited-memory fix;
- assuming interpreter heap covers native TLS allocations;
- treating a blacklist around embedded `exec` as a proven sandbox: runtime/compiler optimizations can bypass metering, and code may share secrets-bearing process state;
- treating immediate post-success logging as exactly-once protection across the acceptance/checkpoint gap;
- auto-formatting established storage or rebooting repeatedly instead of diagnosing corruption;
- clearing a server rate-limit hold during firmware migration;
- treating a live host-backed emulator, successful upload, stale UI label, or unavailable serial stream as proof of standalone device health.
