# ElevenLabs CLI Demo

A showcase of the [ElevenLabs CLI](https://github.com/elevenlabs/cli) (v1.0.0) driven entirely from inside Claude Code — no browser, no GUI. Every audio file here was generated, verified, and published from a terminal.

- **CLI docs:** https://elevenlabs.io/docs/eleven-agents/operate/cli
- **Launch video:** https://www.youtube.com/watch?v=MuORAEVtJIQ
- **How this was built (warts and all):** [DEVELOPMENT-JOURNEY.md](DEVELOPMENT-JOURNEY.md)

## 🎧 Listen

**Inline player: https://az9713.github.io/elevenlabs-cli-demo/** (GitHub Pages)

Or click any file — GitHub shows a play button on the file page:

| File | What it is | Command |
|---|---|---|
| [joke.mp3](audio/joke.mp3) | A British narrator tells a joke about this CLI | `text-to-speech convert` |
| [dialogue.mp3](audio/dialogue.mp3) | Two voices, emotion tags (`[excited]`, `[whispering]`) | `text-to-dialogue convert` |
| [ovation.mp3](audio/ovation.mp3) | Applause inside a humming server room | `text-to-sound-effects convert` |
| [finale.mp3](audio/finale.mp3) | Chiptune fanfare melting into smooth jazz | `music compose` |
| [hello.mp3](audio/hello.mp3) | "Hello from the new ElevenLabs CLI." | `text-to-speech convert` |
| [door.mp3](audio/door.mp3) | A castle door creaks open | `text-to-sound-effects convert` |
| [track.mp3](audio/track.mp3) | Lo-fi track for a rainy afternoon | `music compose` |

## The commands

```bash
# Cast a voice from the catalog (JMESPath filtering built in)
elevenlabs voices search --search "british dramatic narrator" \
  --query 'voices[:3].{id:voice_id,name:name}'

# Text to speech
elevenlabs text-to-speech convert --voice-id 4dZr8J4CBeokyRkTRpoN \
  --text "A developer walks into a bar..." --output joke.mp3

# Two-speaker dialogue with emotion tags
elevenlabs text-to-dialogue convert --json '{"inputs":[
  {"text":"[excited] Did you hear? The entire ElevenLabs API is a command line now!","voice_id":"4dZr8J4CBeokyRkTRpoN"},
  {"text":"[skeptical] Sure. And I suppose it tells jokes too?","voice_id":"wAGzRVkxKEs8La0lmdrE"},
  {"text":"[whispering] It just did. You are inside one.","voice_id":"4dZr8J4CBeokyRkTRpoN"}]}' \
  --output dialogue.mp3

# Sound effects and music
elevenlabs text-to-sound-effects convert --text "applause inside a server room" \
  --duration-seconds 5 --output ovation.mp3
elevenlabs music compose --prompt "8-bit victory fanfare melting into smooth jazz" \
  --music-length-ms 15000 --output finale.mp3

# Verify your own output: transcribe the speech back
elevenlabs speech-to-text convert --file joke.mp3 --model-id scribe_v1 --query '{text:text}'

# Agents as code: pull, edit the JSON, preview, push
elevenlabs agents pull --all --yes
elevenlabs agents push --dry-run
```

## Agents as code

[`agent_configs/my-first-agent.json`](agent_configs/my-first-agent.json) is a real agent config pulled with `elevenlabs agents pull` — the agent's entire personality as version-controlled JSON. [`examples/default-template.json`](examples/default-template.json) is the output of `elevenlabs agents templates show default`: every available field at its documented default.

## Notes

- The CLI is a single Rust binary (33 modules — the full ElevenLabs API). Do **not** confuse it with the older npm package `@elevenlabs/cli` (agents-only). Install via brew, scoop, or the curl script from the [repo](https://github.com/elevenlabs/cli).
- Video generation (`elevenlabs flows video create --json '{"model_id":"bytedance-seedance-v2-mini","prompt":"..."}'`) is command-complete here but needs a Creator+ plan; the starter tier returned `402 paid_plan_required`.
- Agent-friendly by design: `--dry-run` everywhere, `--schema` for machine-readable help, errors returned twice (JSON + a "did you mean" hint), idempotent push/pull, and `generate-skills` writes 31 SKILL.md files so a coding agent can learn the whole surface offline.
