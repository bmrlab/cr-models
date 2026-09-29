# cr-models

Agent plugins for using models on [CR](https://creative-reasoning.com) (creative-reasoning), a multi-model API platform. Works with Claude Code and Codex.

| Plugin | What it does |
|---|---|
| `cr-media` | Generate and edit images and videos: text-to-image, image editing, text-to-video, image-to-video (first frame, first + last frame), reference-to-video (put a character into a new scene), video editing. |

Each model's notes (which parameters work, which fail, which are silently ignored, output sizes, timing) come from real test runs, not from docs.

## Install

### Claude Code

```bash
claude plugin marketplace add bmrlab/cr-models
claude plugin install cr-media@cr
```

Update later with `claude plugin update cr-media@cr`, or turn on auto-update for the `cr` marketplace in `/plugin` → Marketplaces.

### Codex

```bash
codex plugin marketplace add bmrlab/cr-models
codex plugin add cr-media@cr
```

Update later with `codex plugin marketplace upgrade cr`.

### Manually

```bash
git clone https://github.com/bmrlab/cr-models.git
ln -s "$PWD/cr-models/plugins/cr-media/skills/cr-media" ~/.claude/skills/cr-media   # Claude Code
ln -s "$PWD/cr-models/plugins/cr-media/skills/cr-media" ~/.agents/skills/cr-media   # Codex
```

Update with `git pull`.

## API key

The scripts read `CR_API_KEY` from the environment, or from `~/.config/cr/env`:

```bash
mkdir -p ~/.config/cr
printf 'CR_API_KEY=%s\n' '<your key>' > ~/.config/cr/env
chmod 600 ~/.config/cr/env
```

## Requirements

`curl`, `jq`, `ffprobe` (from ffmpeg), `file`, `base64`. Tested on macOS.

## Found something the notes don't match?

Open an issue with the model, the parameters, the task id and what you saw.
