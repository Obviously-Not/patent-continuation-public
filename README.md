# patent-continuation

> **This is a distribution repository.** It carries the documentation and the
> released binaries. The source lives in a private repository and is not
> published here, so there is nothing to build or send a pull request against:
> every file here is generated from the source repo on each release and is
> overwritten by the next one. Bug reports and feature requests are welcome in
> this repo's issue tracker.

> Continuation-claim **drafting** pipeline for licensed patent practitioners,
> shipped as a **Go binary**. A specification plus a set of filed (parent) claims
> goes in; drafted continuation claims come out, refined by a cross-family critic.
> All inference runs locally by default.
>
> Intended audience: **licensed patent attorneys and agents** who run it as a
> drafting aid under their own professional judgment.

## What it is

A Go binary. A specification plus the parent claims go in; drafted continuation
claims come out. All inference is local by default (Ollama), or through any
OpenAI-compatible endpoint with your own key. Nothing is sent anywhere you do
not configure, and there is no server.

| Piece | Role |
|---|---|
| `go/main.go` | CLI: `draft`, `critique`, `judge`, `panel`, `serve`, `models`, `version`, `license`, `activate`, `upgrade` |
| `go/internal/server/` | HTTP server for the local web UI (runs on localhost:9473) |
| `go/web/` | Alpine.js SPA served by the binary (no build step). HTML/CSS are embedded via go:embed; Alpine itself loads from a pinned CDN with a subresource-integrity hash, so the web UI needs network access on first load and is not air-gapped |
| `go/internal/pipeline/draft.go` | Spec analysis, target extraction, drafting, and the verify->revise loop |
| `go/internal/pipeline/critic.go` | Cross-family critic that flags per-claim defects for the revise loop |
| `go/internal/pipeline/judge.go` | Claim-quality rubric judge with spec-anchor validation |
| `go/internal/pipeline/panel.go` | Multi-model scoring panel behind the `panel` subcommand (R&D; cannot rank near-equals) |
| `go/internal/license/` | Offline Ed25519 license verification (multiple keys, multiple capability tiers) |
| `go/internal/llm/` | Provider layer: local Ollama plus an OpenAI-compatible remote |

## Setup

You need the binary (download it from the latest release, or see below to build
it), and **at least one inference backend**:
local **Ollama** (private: nothing leaves your machine) or a remote **OpenRouter**
key (faster and stronger, but the spec is sent to a third party). You can set up
both and choose per run with `--model`. Keys are read from the environment only,
never from a flag and never from a `.env` file the binary loads for you, so you
must `export` them in your shell (CP3).

### Option A: local models with Ollama (private)

1. **Install Ollama** from https://ollama.com/download (macOS, Linux, Windows).
   On macOS you can also `brew install ollama`.
2. **Start the server.** The desktop app starts it for you; otherwise run
   `ollama serve`. It listens on `http://localhost:11434`. Point the tool at a
   different host (for example a beefier machine on your LAN) with
   `export OLLAMA_BASE_URL=http://that-host:11434`.
3. **Pull a model before you run.** The binary does **not** download models for
   you, and it assumes **no default model**: name one with `--model` or a
   `*_MODEL` env var, or the run fails fast with guidance. Pull one first:

   ```bash
   ollama pull qwen3:32b         # capable general model; 41k window
   ollama pull qwen3-coder:30b   # 262k window: fits a real (124k-char) spec
   ```

   Context is the first filter: a spec is ~1 token per 4 characters and the whole
   thing goes into the prompt, so the model's context window must exceed the spec.
   The tool sizes the request's context window to the prompt automatically, so any
   model gets its full window with no per-model tuning; you only need to pick a
   model whose window fits your spec. **Read [`docs/models.md`](docs/models.md)
   before choosing a model for real work** (which models fit real specs, which
   draft well, which fail outright). There is no single best local model, it is a
   tradeoff between window size and drafting quality.
4. **Verify:** `ollama list` shows the model you pulled; `./continuation-drafter
   models` shows the configuration the tool resolved.

### Option B: remote models with OpenRouter (faster, stronger)

1. **Create an API key** at https://openrouter.ai/keys and add credit. A single
   draft makes several model calls, so check your balance at
   https://openrouter.ai/credits before a big run.
2. **Export it:** `export OPENROUTER_API_KEY=sk-or-...`
3. **Select a remote model** with the slash-slug form: `openai/gpt-5.5`,
   `anthropic/claude-opus-4-8`, `moonshotai/kimi-k2.5`, or any OpenRouter slug.
   Override the endpoint (rare) with `OPENROUTER_BASE_URL`; it defaults to
   `https://openrouter.ai/api/v1`.

**Confidentiality:** a remote model sends the specification to a third party.
That is fine for published patents and **not** for anything unpublished or
privileged, use the local path (Option A) for those. The "nothing leaves your
machine" guarantee belongs to the local path only.

## Run it

Get the binary once, then run it from wherever your files are (paths resolve
from your current directory).

```bash
# Download a prebuilt binary (pick your OS and architecture: darwin-arm64,
# darwin-amd64, linux-amd64, linux-arm64, or windows-amd64.exe).
gh release download --repo Obviously-Not/patent-continuation-public \
  --pattern 'continuation-drafter-darwin-arm64'
chmod +x continuation-drafter-darwin-arm64
mv continuation-drafter-darwin-arm64 continuation-drafter
```

**Draft** (the main command). A spec plus the parent claims in, drafted
continuation claims out on stdout; live progress prints to stderr:

```bash
# Local model (nothing leaves your machine)
DRAFT_MODEL=qwen3:32b ./continuation-drafter draft \
  --spec ../fixtures/micro/micro_spec.txt \
  --parent ../fixtures/micro/micro_parent_claims.txt

# Remote model via OpenRouter (spec leaves your machine; published material only)
OPENROUTER_API_KEY=... ./continuation-drafter draft \
  --spec spec.txt --parent parent.txt \
  --model openai/gpt-5.5 --revise-loops 2
```

**Critique** an existing draft on its own (per-claim 112(a)/112(b) defects with
fixes), independent of the drafting loop:

```bash
./continuation-drafter critique --spec spec.txt --parent parent.txt \
  --draft claims.txt --model anthropic/claude-sonnet-5 --json
```

**Other commands:** `judge` scores one draft against the rubric, returning
per-claim verdicts with spec-anchor validation (T1) and honest-null: a claim it
cannot anchor to the spec comes back `indeterminate`, not a low score (R&D; the
score is not a ranking, the support audit is the useful part). `panel` scores
candidate drafts with a multi-model judge (R&D: it separates gross differences
but cannot rank near-equals, so read it as a spread, not a ranking); `models`
prints the resolved model configuration; `version` prints build info. Run
`<command> --help` for every flag.

## Web UI

The binary includes a local web UI. Start it with `serve`:

```bash
./continuation-drafter serve           # opens http://localhost:9473 in your browser
./continuation-drafter serve --port 8080 --no-browser
```

The web UI provides full CLI parity with a graphical interface:
- **Draft tab**: Generate continuation claims with real-time SSE progress
- **Critique tab**: Analyze existing claims for 112(a)/112(b) defects with suggested fixes
- **Judge tab**: Score claims with spec-anchor validation (T1/T4 semantics)
- **Panel tab**: Compare multiple drafts with multi-model scoring
- File drag-drop or paste for spec, parent claims, and existing drafts
- Model selection (from configured models)
- Light/dark theme (follows system preference)
- License key activation modal

The server binds to `127.0.0.1` only (localhost, not network-accessible). Your inputs
(spec, parent claims, drafts) stay on your machine; the only place they are sent is the
LLM provider you configure. One honest caveat: the web UI loads the Alpine.js library
from a pinned CDN (with an integrity hash) when the page opens, so the browser makes one
request to that CDN on load. That request carries no spec or claim data, but it does mean
the **web UI needs network access on first load and is not fully air-gapped**. The CLI has
no such dependency and runs fully offline (with a local Ollama model).

## License keys

Keys are Ed25519-signed, offline-verified, and stored locally in
`~/.continuation-drafter/config.json`. Multiple keys can be activated; each key's tier is
verified and reported. **As of 2026-07-21 the tier is reported but does not yet gate any
feature: the free/paid paywall is deferred, so all features are available today.**

```bash
./continuation-drafter license             # show license status and capabilities
./continuation-drafter activate --key <your-key>   # activate a key
./continuation-drafter upgrade             # open the purchase page in your browser
```

**Note:** There is currently no production public key embedded in dev builds, so
license verification is skipped. The license system is implemented and tested but
becomes load-bearing only in release builds with an embedded public key.

### The knobs that matter

- `--model` selects the model and the routing (no default is assumed; name one).
  A **colon** tag (`qwen3:32b`) is a local Ollama model; a **slash** slug
  (`openai/gpt-5.5`,
  `anthropic/claude-opus-4-8`) or a recognized remote prefix (`claude-`, `gemini-`,
  `grok-`, `gpt-`, `chatgpt`, or an o-series name) routes to OpenRouter (a bare
  `gpt-5.5` is rewritten to `openai/gpt-5.5`). Ambiguous bare names that exist in
  both worlds (`llama`, `qwen`, `deepseek`, `mistral`) default to LOCAL; add the
  `vendor/name` slash form to force one remote. If a bare name is not a pulled
  local model, the error hints at the remote form. See
  [`docs/models.md`](docs/models.md).
- `--revise-loops N` (default 3) is how many critic/revise passes run; each pass
  costs two more model calls. `0` gives the raw first-pass draft.
- **The critic runs on its own model.** By default it is the draft model; set
  `CRITIC_MODEL` (or `critique --model`) to make the revise loop cross-family.
  Gotcha worth stating: if you draft on a large-window remote model but leave the
  critic on a small local default, the critic can choke on a big spec. Point the
  critic at a model that also fits the spec.
- `--max-tokens` / `MAX_COMPLETION_TOKENS` (default 16000) is the per-call
  completion cap. `--timeout` (default 30m for `draft`) and
  `CD_REQUEST_TIMEOUT_SECONDS` (per-request) bound long runs; large specs on slow
  models need both raised.
- Keys come from the environment only, never a flag. The binary reads exactly
  one key, `OPENROUTER_API_KEY`; all remote inference goes through OpenRouter, so
  no per-vendor key is used (CP3).
- `--json` emits a structured envelope on stdout instead of plain text.

**Model choice matters more than it looks, and context is the first filter.**
A specification is ~1 token per 4 characters and the whole thing goes into the
prompt, so a model whose window is smaller than the spec **silently truncates
its input** and drafts from a spec it never fully saw. A 124k-character spec is
~31k tokens, which rules out every 32k-40k model no matter how well it drafts.

Measured results for the models available locally, including which ones fail
outright, are in [`docs/models.md`](docs/models.md). Thinking models (qwen3,
glm, deepseek-r1) are handled: the binary disables thinking so the completion
budget buys claim text rather than discarded reasoning.

Remote models via OpenRouter work too (`--model anthropic/claude-opus-4-8` with
`OPENROUTER_API_KEY` set), but the spec then leaves your machine. Use a local
model for anything unpublished or privileged. Note the local path costs more than
speed: on real specs, local models draft well-grounded but strategically flatter
claims than frontier models, so treat a local draft's claim aim as a first pass.

## Honest maturity

This is **R&D-grade**, not a finished product. The originating experiment's
conclusions still hold: the pipeline drafts real, well-formed, spec-grounded
continuation claims, but a **single AI judge cannot reliably rank drafters**, the
revise loop is a modest positive, and only a multi-model panel gives a stable
read. Treat output as drafting *input* for a practitioner, never as filing-ready
and never as a legal determination.

The licensing system's **verification and reporting are implemented; feature gating is
deferred**. There is no production public key embedded, so all features are available in
current builds. Even once a production key is embedded, keys will verify and report a tier
but will not gate any feature until the paywall is built (deferred by owner decision, R3).

## License

Proprietary. All rights reserved. Not open source. The binary is distributed
freely; the source is not published.
