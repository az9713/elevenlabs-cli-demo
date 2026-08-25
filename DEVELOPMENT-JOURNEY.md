# Development Journey — "ElevenLabs CLI Demo" from inside Claude Code

**Date:** 2026-08-24
**Deliverable:** https://github.com/az9713/elevenlabs-cli-demo — 7 generated audio files, agent configs as code, this document
**Brief:** "Read https://elevenlabs.io/docs/eleven-agents/operate/cli."
**Final artifacts:** `audio/*.mp3` (7 files), `agent_configs/my-first-agent.json`, `examples/default-template.json`

## 1. The brief — a docs read that became a full showcase

The session started with three words of instruction: read the CLI docs page. It grew, one user message at a time, into: learn every command, install the CLI, run examples, hear the agents talk, generate a joke, a two-voice sketch, sound effects, and music — all without leaving the Claude Code terminal — then publish the whole thing to GitHub.

Nothing about this arc was planned at minute zero. The working folder contained only `README.txt` (the user's notes on a launch video), `transcript.txt` (the video's transcript), and a `.env` with an API key. Those notes turned out to be the most important files in the session — they caught my biggest mistake.

## 2. Cold start — reading three sources that disagreed

I read the docs page (`/docs/eleven-agents/operate/cli`), then its `llms.txt` variant. Both described an agents-only CLI: `init`, `auth`, `agents add/push/pull/list/status`, `tools`, `templates`, `widget`, `generate-skills`. Six templates. A JSON project layout.

The user then asked: "do you learn all the commands and their options?" Honest answer: no. Their `README.txt` mentioned `elevenlabs text-to-speech convert` and `elevenlabs music compose` — commands the docs page never named. First signal that the docs page was a partial view.

## 3. Design decisions

- **npm vs Scoop vs curl for install.** The docs listed brew/scoop/curl; their CI example used `npm install -g @elevenlabs/cli`. I chose npm because Node was already present. **This was the session's core mistake** — see section 4.
- **API key via `.env`, never in the keyring or chat.** Mid-session the user sent: "do not reveal secrets." All key output since is redacted; the CLI reads `ELEVENLABS_API_KEY` from the environment directly.
- **`--dry-run` for anything that writes to the account.** The agent-edit demo stopped at `[DRY RUN] Would update agent: my first agent`. Nothing was ever pushed to the user's ElevenLabs account in the entire session.
- **Verify by observed effects, not exit codes.** Every mp3 was played through `ffplay -autoexit` after generation; the TTS output was transcribed back with speech-to-text and string-compared to the input.
- **Publish only scrubbed content.** One of the ten pulled agent configs contained personal bio information in its prompt. The public repo carries only `my-first-agent.json` (scanned clean) plus a generated `examples/default-template.json` — the full parameter schema with zero account data.

## 4. The wrong CLI — same name, different product

I installed `@elevenlabs/cli` 0.5.6 from npm. It worked: `agents init`, `auth whoami`, `agents pull` (10 agents downloaded), `agents list`, `agents status`, `templates show`, `widget`. I confidently reported the CLI's surface: 5 modules, no audio commands.

Then the user pushed back: "Read README.txt and transcript.txt. there is text-to-speech. please confirm."

The transcript (a launch video) said at 0:47: "agents, audio isolation, dubbing, flows, music, speech engine, speech to text, text to speech…" — and at 0:08: "every endpoint we publish on the OpenAPI spec becomes a subcommand."

Two products share one name:

| | npm `@elevenlabs/cli` 0.5.6 | Rust binary 1.0.0 (brew/scoop/curl) |
|---|---|---|
| Scope | agents-as-code only | the entire ElevenLabs API |
| Modules | 5 | 33 |
| Size | 97 npm packages | one 5.4 MB zip |

The docs page sits under the *Agents* section and documents only the agents half — accurate but incomplete. Its CI example's npm line pointed at the old package. The repo README (`github.com/elevenlabs/cli`) never mentions npm at all.

**Rule learned: when a doc page and a primary source (the launch video) disagree about a tool's surface, install the tool and ask it — `--help` is the only authority that cannot be stale.**

The fix: `npm uninstall -g @elevenlabs/cli` (exit code 1 despite "removed 97 packages" — the uninstall succeeded; the nonzero exit was noise), then `scoop bucket add elevenlabs https://github.com/elevenlabs/scoop-bucket && scoop install elevenlabs` → `elevenlabs 1.0.0`, 33 modules. All old generated files were deleted and re-created with the new binary; the user asked for this twice, and file timestamps (configs written one minute *after* the scoop install) proved the redo was already done — I re-ran it anyway for certainty.

## 5. Tools and features used

| Tool / command | What it did this session |
|---|---|
| `WebFetch` | Read the docs page and `llms.txt` |
| `curl` on `cli.md` | Got the raw markdown when WebFetch summarized too aggressively |
| `npm install/uninstall -g` | Installed, then removed, the wrong CLI |
| `scoop bucket add` + `scoop install` | Installed the real CLI (1.0.0) |
| `elevenlabs agents init/pull/list/status/push --dry-run` | Agents-as-code round trip; 10 agents pulled read-only |
| `elevenlabs voices search --search --query` | Voice casting via JMESPath, no `jq` |
| `elevenlabs text-to-speech convert` | `hello.mp3`, `joke.mp3` |
| `elevenlabs text-to-dialogue convert --json` | `dialogue.mp3`, two voices, `[excited]`/`[skeptical]`/`[whispering]` tags |
| `elevenlabs text-to-sound-effects convert` | `door.mp3`, `ovation.mp3` |
| `elevenlabs music compose` | `track.mp3`, `finale.mp3` |
| `elevenlabs speech-to-text convert` | Round-trip verification of TTS output |
| `elevenlabs generate-skills` | 31 SKILL.md files — the CLI documenting itself offline |
| `elevenlabs flows video create` | Attempted; blocked by plan tier (section 6) |
| `--spec` / `--schema` | Read the embedded OpenAPI spec to learn the video models offline |
| `ffplay -autoexit` | Played every mp3 as verification |
| `gh` | Created and pushed this public repo |

No subagents; all work inline in one session.

## 6. What went wrong, and the fixes

1. **Wrong CLI installed** (section 4). Fixed by uninstall + scoop install.
2. **`agents widget "Receptionist"` failed**: `Error generating widget: Error: Agent with ID 'Receptionist' not found in configuration`. The command takes an ID, not a name. Fixed by reading the ID from `agents.json`.
3. **`flows video create --params '{...}'` returned 422** with `"loc": ["body"], "type": "missing"`. `--params` fills path/query/header parameters; the request body goes in `--json`. Rule learned: **read `--schema` before the first call, not after the error.**
4. **Video generation returned 402** `paid_plan_required`. The starter tier covers TTS, dialogue, sound effects, music, and STT — not video (`veo-3.1`, `seedance-v2`, `creatify-aurora` are Creator+). The correctly-formed command is in the README for anyone on a higher tier.
5. **`auth status` says `logged_in: false` even while every API call works.** It reports only the OAuth keyring entry; an env-var API key is invisible to it. Cosmetic, but confusing.
6. **Scoop's install output was 117 KB** of unrelated bucket-update noise; the actual result ("'elevenlabs' (1.0.0) was installed successfully!") was the last line.
7. **Near-miss:** I almost told the user the CLI had no audio commands at all — a wrong conclusion their own notes file corrected. The stale docs page nearly became "the truth" because it was the first source read.

## 7. Verification

- Every mp3 played through `ffplay -autoexit` (observed exit 0 after audible playback on the user's machine).
- TTS round trip: `joke.mp3` transcribed back by `scribe_v1` matched the input text word-for-word; only punctuation drifted (quotes added, final period dropped).
- Account safety: `elevenlabs agents list` against the live API confirmed the same 10 agents, same IDs, before and after all file deletions. `pull` is read-only; nothing was pushed.
- The one intended change (`my-first-agent` greeting) exists only in the local JSON and this repo; the dry-run output is the proof it never went live.
- Not verified: video generation output (plan-blocked) and the widget embed in a browser (not attempted).

## 8. Where things stand

- **Live:** this repo, with 7 playable audio files and the GitHub Pages player.
- **Costs:** 9,532 / 30,021 characters used on the starter tier after the whole session (~1,335 characters for all seven showcase generations).
- **Blocked:** `flows video` and `dubbing` need a Creator+ plan. The exact commands are ready in the README.
- **Open option:** push the fun greeting to the live agent with `elevenlabs agents push --agent <id>` — one command, currently parked at dry-run.
