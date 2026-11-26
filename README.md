# ArtcraftBot

A standalone, open-source (MIT) bot platform built for the ArtCraft suite —
ground-up, with no dependency on any hosted agent platform.

Bots live in sandboxed homes with their own identity, memory, and tools.
Autonomy is bounded by AutoCraft-style trust boundaries instead of per-edit
human locks: scoped workspaces, per-bot spend caps, mechanical done-checks,
and a single audit-logged write/promote gate at the edge. Any open bot
harness can occupy a home — OpenCode first, others through a generic
BotAdapter. Every model call passes through one gateway and is recorded in
an append-only, untainted log.

## Status

Under construction. The base lands in this order:

1. **Untainted logger** — every call, exactly as it happened.
2. **Model gateway** — provider routing, per-bot caps, backoff.
3. **BotAdapter** — harness-agnostic runner interface + the OpenCode
   (`serve`) implementation.

The builder brief for the base is in `docs/`. The full platform spec
follows in the same place.

Private during construction; the public release is the founder's call.
