# Hermes Agent Ecosystem — Verified Category Map

Reference for ecosystem discovery tasks targeting Hermes Agent. Captures the
hubs, CLI commands, trust model, and verified category map. Distilled from
live GitHub searches and README verification; not a static directory.

## Hubs and discovery entry points

1. **Official org**: `NousResearch/hermes-agent` — the main repo, plugin catalog
   lives under `plugin-catalog/` (YAML pins with full 40-char commit SHA).
2. **Official docs**: `hermes-agent.nousresearch.com/docs` — platform support
   matrix, install guide, plugin development.
3. **Community plugins**: installed via `hermes plugins install <repo-url-or-slug>`,
   enabled via `hermes plugins enable <id>`. Catalog entries are reviewed + SHA-pinned;
   community (non-catalog) installs show "custom (unreviewed) source" warning.
4. **GitHub topic tags**: `hermes-agent`, `hermes-plugin`, `hermes-desktop`.
   Topic search is noisy — verify README for real integration path before featuring.
5. **Awesome lists**: search `awesome hermes` on GitHub; treat paid featured slots
   as ads.

## CLI commands (verified)

- `hermes plugins install <slug-or-url>` — clone into `$HERMES_HOME/plugins/<id>/`
- `hermes plugins enable <id>` — register in config.yaml
- `hermes plugins validate <path>` — 14+ checks including SDK surface lint
- `hermes plugins doctor <id>` — health check (key, model, run lane)
- `hermes plugins update <id>` — pull catalog-pinned revision
- `hermes gateway run` — start the gateway with enabled platforms

## Trust model

- **Catalog plugins**: reviewed, SHA-pinned, `hermes plugins install <slug>` works
  without warning. Highest trust.
- **Community plugins**: `hermes plugins install <url>` shows "custom (unreviewed)
  source" warning. Verify README for install steps and license before recommending.
- **Plugin structure**: most Hermes plugins follow `plugin.yaml` + `__init__.py` +
  `tools.py` + `cli.py` + optional `dashboard/plugin_api.py` + `desktop/plugin.js`.
  Desktop plugins are uncompiled ESM (no JSX, only `@hermes/plugin-sdk`, `react`,
  `react/jsx-runtime` resolve).
- **Profiles**: separate `HERMES_HOME` per profile; plugins don't inherit across
  profiles. Symlink or junction each profile to a shared clone.

## Verified category map (Hermes voice/full-duplex ecosystem)

### Full-duplex voice (Desktop)

| Project | Stars | Engine | Install | License |
|---------|-------|--------|---------|---------|
| Synero/hermes-live-voice | 39 | GPT-Live-1 (Codex sub) | `hermes plugins install hermes-live-voice` | MIT |
| skappafrost/hermes-gemini-live | 4 | Gemini Live | `hermes plugins install skappafrost/hermes-gemini-live` | MIT |
| Julientalbot/hermes-xai | 7 | xAI Grok Voice | `hermes plugins install https://github.com/Julientalbot/hermes-xai` | MIT |

### Full-duplex voice (Messaging platforms)

| Project | Stars | Platform | Engine | License |
|---------|-------|----------|--------|---------|
| Capslockb/hermes-live-discord-agent-plugin | 24 | Discord | Gemini Multimodal Live | none (no LICENSE) |
| ashwanthbalakrishnan5/hermes-whatsapp-voice | 108 | WhatsApp | Gemini Live | MIT |
| drewdah/mouthpiece | 1 | Windows tray (LiveKit) | Hermes TTS provider | MIT |

### Speech-to-speech / voice mode

| Project | Stars | Modes | License |
|---------|-------|-------|---------|
| mrjonathanm/hermes-s2s | 0 | cascaded/realtime/s2s-server; Gemini Live, GPT-4o Realtime, local | Apache-2.0 |

### Gateway plugins (messaging)

| Project | Stars | Platform | License |
|---------|-------|----------|---------|
| ii999/hermes-napcat | 0 | QQ (OneBot v11) | MIT |

### Curiosity / experimental

| Project | Stars | Notes |
|---------|-------|-------|
| smaniktahla/nina-full-duplex-voice-AI | 1 | Local full-duplex voice assistant; Home Assistant + Hermes tools |
| justinruaya123/hermes-full-duplex | 0 | Personal plugin substituting gpt-live-1 |
| bluecli/hermes-voice | 0 | Fish Audio full-duplex voice calls |
| SirioForel/hermes-voice-proxy | 0 | Real-time bidirectional voice proxy for IT meetings |

## Search patterns that work

- `"hermes" "full duplex"` — surfaces the most relevant repos
- `"hermes" "live voice"` — catches desktop voice plugins
- `"hermes" "voice plugin"` — catches messaging platform bridges
- `topic:hermes-agent` — broad but noisy; filter by description mention
- `topic:hermes-plugin` — often returns 0; fall back to description search
- `"hermes agent" in:description` — good secondary sweep

## Common pitfalls (Hermes-specific)

- **Many repos are 0-star or unlicensed** — flag in digest, don't present as adoptable.
- **Topic tags are noisy** — `topic:hermes-agent` returns the main repo and unrelated projects; verify README for real integration.
- **Profile isolation** — plugins don't inherit across profiles; must symlink per profile.
- **Desktop half requires restart** — plugin API routes mount at web-server import; Desktop half is a copy under `desktop-plugins/` refreshed only at app start.
- **No standalone LICENSE** — several community plugins (e.g. Capslockb's) have no LICENSE file; do not assume reuse rights.
- **Non-English repos** — some Chinese-language projects (ii999/hermes-napcat) ship working code; note language in digest.
- **Prebuilt binaries** — some plugins (hermes-whatsapp-voice) ship Rust sidecars via GitHub releases; check platform support (linux amd64/arm64, macOS arm64).
