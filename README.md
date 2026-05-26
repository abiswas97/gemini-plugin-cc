# Gemini plugin for Claude Code

> [!IMPORTANT]
> **`gemini-cli` deprecation notice — June 18, 2026 cutoff**
>
> Google [announced at I/O 2026](https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/) (May 19) that `gemini-cli` is being replaced by **Antigravity CLI (`agy`)**. Free-tier and AI Pro/Ultra users will stop receiving requests on **June 18, 2026**. After that date, this plugin will silently fail for those user tiers. Gemini Code Assist for Enterprise and Vertex AI authenticated users are not affected.
>
> **Migration path:** [`antigravity-plugin-cc`](https://github.com/MarcosNahuel/antigravity-plugin-cc) is a Claude Code plugin that wraps `agy` instead of `gemini-cli`, with a similar slash-command shape plus four extras (`/antigravity:record` browser walkthrough recording to `.webm` + MP4, `/antigravity:scrape` structured URL extraction, `/antigravity:doc-to-md` PDF/docx/image to Markdown, `/antigravity:design-review` UX audit).
>
> ```text
> /plugin marketplace add MarcosNahuel/antigravity-plugin-cc
> /plugin install antigravity@marcosnahuel-antigravity
> ```

Use Gemini from inside Claude Code for code reviews or to delegate tasks to Gemini.

Based on [`openai/codex-plugin-cc`](https://github.com/openai/codex-plugin-cc), adapted for the Gemini CLI. Also adds `/gemini:task` for direct task delegation.

## What You Get

- `/gemini:review` for a normal read-only Gemini review
- `/gemini:adversarial-review` for a steerable challenge review
- `/gemini:rescue`, `/gemini:status`, `/gemini:result`, and `/gemini:cancel` to delegate work and manage background jobs
- `/gemini:setup` to verify Gemini CLI is ready and manage the review gate
- `/gemini:task` for quick one-off task delegation

## Requirements

- **Gemini CLI** installed and authenticated — [install guide](https://developers.google.com/gemini-cli)
- **Node.js 18.18 or later**
- **Google API key** or Application Default Credentials — [create an API key](https://aistudio.google.com/app/apikey) or run `gcloud auth application-default login`

## Install

Add the marketplace in Claude Code:

```bash
/plugin marketplace add abiswas97/gemini-plugin-cc
```

Install the plugin:

```bash
/plugin install gemini@abiswas97-gemini
```

Reload plugins:

```bash
/reload-plugins
```

Then run:

```bash
/gemini:setup
```

`/gemini:setup` will tell you whether Gemini is ready. If Gemini is missing, refer to the [Gemini CLI installation guide](https://developers.google.com/gemini-cli).

If Gemini is installed but not authenticated, set up your credentials:

```bash
!gcloud auth application-default login
```

Or set the `GOOGLE_API_KEY` environment variable with a key from [AI Studio](https://aistudio.google.com/app/apikey).

After install, you should see:

- the slash commands listed below
- the `gemini:gemini-rescue` subagent in `/agents`

## Configuration

| Variable | Purpose |
|---|---|
| `GOOGLE_API_KEY` | API key from AI Studio. Alternative to `gcloud auth application-default login`. |
| `GEMINI_ACP_INIT_TIMEOUT_MS` | Override the 30s ACP initialize timeout. Bump it if Gemini CLI cold-start is slow on your machine (e.g. Homebrew installs that fall back to a file-based keychain). |

## Commands

| Command | Description |
|---------|-------------|
| `/gemini:review` | Code review on current work or branch. Supports `--wait`, `--background`, `--base <ref>`. |
| `/gemini:adversarial-review` | Challenges design choices, not just bugs. Same flags as review plus custom focus text. |
| `/gemini:rescue` | Delegate a task to Gemini via subagent. Supports `--background`, `--resume`, `--fresh`, `--model <pro\|flash\|pro-3\|flash-3>`. |
| `/gemini:task` | Quick one-off task delegation. Supports `--background`, `--model`. |
| `/gemini:status` | Show running and recent jobs. |
| `/gemini:result` | Show output for a finished job. Includes session ID for `gemini resume`. |
| `/gemini:cancel` | Cancel an active background job. |
| `/gemini:setup` | Check Gemini CLI readiness. Toggle review gate with `--enable-review-gate` / `--disable-review-gate`. |

## Typical Flows

```bash
# Review before shipping
/gemini:review --background
/gemini:status
/gemini:result

# Hand a problem to Gemini
/gemini:rescue investigate why the build is failing in CI

# Challenge your design
/gemini:adversarial-review --base main question the caching strategy
```

## Model Selection

The `--model` flag accepts shortcuts or full model names:

- `flash` → `gemini-2.5-flash` (default)
- `pro` → `gemini-2.5-pro`
- `flash-3` → `gemini-3-flash-preview`
- `pro-3` → `gemini-3-pro-preview`

If omitted, defaults to `gemini-2.5-flash`. Any concrete model name (e.g. `gemini-3.1-pro-preview`) is passed through as-is.

## Review Gate

When enabled (`/gemini:setup --enable-review-gate`), the plugin runs a targeted Gemini review on Claude's response before stopping. If issues are found, the stop is blocked so Claude can address them first.

> [!WARNING]
> The review gate can create a long-running Claude/Gemini loop. Only enable it when actively monitoring the session.

## Development

```bash
pnpm test        # run tests
pnpm run ci      # type check + lint + test
pnpm run lint:fix  # auto-fix lint issues
```

### Project Structure

```
plugins/gemini/
├── .claude-plugin/plugin.json    # Plugin metadata
├── agents/                       # Gemini rescue subagent
├── commands/                     # Slash commands (.md)
├── hooks/                        # Session lifecycle + review gate hooks
├── prompts/                      # Prompt templates
├── schemas/                      # Output JSON schemas (review, errors)
├── scripts/                      # Companion CLI + runtime modules
└── skills/                       # Claude Code skills
```

## License

Apache 2.0 — see [LICENSE](LICENSE).
