# ReThink Home Assistant add-on repository (unofficial fork)

Home Assistant add-on packaging for [jmcjm/rethink](https://github.com/jmcjm/rethink) —
a fork of [anszom/rethink](https://github.com/anszom/rethink), the fully-local cloud
server for LG ThinQ devices. The fork tracks upstream and adds device handlers not
(yet) merged upstream: the RH90V9_WW heat-pump dryer and extended remote control for
the Y_V8_Y___W.B32QEUK washer (EU).

This is **not** an official anszom/rethink release and is not affiliated with upstream.

## Installation

1. Home Assistant → Settings → Add-ons → Add-on Store → ⋮ → **Repositories**
2. Add `https://github.com/jmcjm/rethink-ha-addon`
3. Install **ReThink Cloud (unofficial)** and read its Documentation tab before first start.

[![Add repository to my Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fjmcjm%2Frethink-ha-addon)

---

## Why this fork exists

Fork of [jmcjm/rethink-ha-addon](https://github.com/jmcjm/rethink-ha-addon),
retargeted to build [forevaclevah2/rethink](https://github.com/forevaclevah2/rethink)
so it includes a handler for **`2REF11EBIR__4`** — the LG LF21G6200S, a US 3-door
counter-depth refrigerator that upstream `anszom/rethink` does not yet recognise.

That model reports a **65-byte status body** where `2REF11EIDA__4` expects 68, so
its `10EB` frame was dropped by an exact-length check and the appliance appeared
as `thinq2 device type 2REF11EBIR__4 unknown` with no entities.

The handler, the frame capture it was derived from, and instructions for
re-deriving it are documented in `docs/2REF11EBIR__4.md` in the source fork
(branch `add-2REF11EBIR__4`).

### What was changed here

Only `rethink/Dockerfile` (the `RETHINK_REPO` / `RETHINK_REV` build args) and the
name/version/description in `rethink/config.yaml`. Everything else is jmcjm's.

### Rebuilding after a change

`RETHINK_REV` is pinned to a commit on the `add-2REF11EBIR__4` branch, not master,
which is why the clone in the Dockerfile is not shallow. To pick up new work:
push to that branch, update `RETHINK_REV` to the new SHA, bump `version:` in
`rethink/config.yaml` (the Supervisor only offers an update when the version
string changes), then Rebuild the add-on in Home Assistant.

### Migrating the CA

rethink generates a CA on first run and provisioned appliances are pinned to it.
Installing this as a *new* add-on gives it a *fresh* CA, which would orphan any
already-provisioned device and force another SoftAP round. Carry the old CA over
via the `ca_key_pem` / `ca_cert_pem` options before first start. They can be
recovered from a partial Supervisor backup of the previous add-on
(`config/ca.key`, `config/ca.cert`).

### Upstream

The handler is written to be upstreamable to `anszom/rethink` as-is. If it lands
upstream, retarget `RETHINK_REPO` back to `anszom/rethink` and drop this fork.
