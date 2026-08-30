# Choosing a model

Measured on 2026-07-19 against the public micro fixture
(`fixtures/micro/`, 3.7KB) with `--revise-loops 0`, so this ranks **raw
drafting**, not the critic repairing it. Every model got the same spec, the
same prompt, and one attempt.

Treat this as a screen, not a ranking: it says who is disqualified, not who is
best. Claim quality beyond form needs a practitioner's eye.

## Results

| model | claims | indep | proper transition | structured body | form defects | secs |
|---|---|---|---|---|---|---|
| `qwen3:32b` | 15 | 4 | 7 | 4 | 0 | 194 |
| `qwen3-coder:30b` | 10 | 5 | 9 | 4 | 0 | 32 |
| `devstral-small-2:24b` | 8 | 4 | 7 | 3 | 0 | 72 |
| `qwen3:8b` | 8 | 3 | 5 | 3 | 0 | 31 |
| `deepseek-r1:14b` | 7 | 3 | 4 | 0 | 0 | 110 |
| `codestral:22b` | 4 | 2 | 4 | 4 | 4 | 76 |
| `glm-4.7-flash` | FAILED | | | | | 12 |

"Proper transition" counts claims using `comprising`/`consisting of`.
"Structured body" counts claims with semicolon-delimited elements.
"Form defects" counts claims using bullet hyphens inside the claim body, which
is not valid US claim form.

## A second axis: structured-JSON validity for judge/critique/panel (2026-07-21)

The table above ranks **drafting**, whose output is plain claim text. `judge`,
`critique`, and `panel` are different: they require the model to emit strict,
schema-valid JSON, and a model that drafts acceptably can still fail these.

Measured 2026-07-21 against the same micro fixture, via the web UI:

| model | draft | judge | critique |
|---|---|---|---|
| `qwen2.5-coder:7b` | ok (3 claims) | FAILED (emitted claim number `1.1`, not an int) | FAILED (no parseable JSON) |

Takeaway: a small local coder model may pass the drafting screen yet be unusable
for scoring/critique because it cannot hold the JSON contract. When judge/critique/
panel return "unparseable JSON", suspect the model before the pipeline. This is a
known limitation, not a bug; the tool reports the parse failure honestly and the
UI surfaces it as an error rather than fabricating a result. Use a stronger model
(a larger local model or a remote model via OpenRouter) for the scoring surfaces.

## Spec size is the first filter, before any model question

> **Updated 2026-07-20:** the fixed 160k-char refusal below is superseded by a
> model-aware ceiling. The tool now detects each model's real context window
> (Ollama `/api/show`, OpenRouter `/models`) and allows any spec that fits it,
> minus the output budget and a 10% margin. `encrepo` (383k chars) and the other
> large specs draft on a 1M-context model; they are refused only on a model too
> small to hold them, and refused in tokens against that model's window. The
> fixed 160k value survives ONLY as the fallback when a window cannot be
> detected.
> The table below documents the OLD behavior for reference. Large specs were
> subsequently confirmed to draft at the same quality as small ones (no
> lost-in-the-middle collapse); see the findings doc.

The tool refuses outright above `SpecFocusThreshold` (160,000 chars, about 40k
tokens) in `go/internal/pipeline/draft.go`:

    specification is N chars, above the 160000-char focus threshold:
    retrieval-focused grounding is not implemented in this build, so drafting
    would be grounded in a silently truncated spec

This is a fail-loud guard, and it is correct: without retrieval-focused
grounding (the accepted, unimplemented coverage gap), drafting from a larger
spec means drafting from a silently truncated one. Refusing beats pretending.

But it is a hard product limit and it bites early. **Five of the ten specs in
this repo's own public corpus exceed it**, for every model, regardless of that
model's context window:

| spec | chars | |
|---|---|---|
| `networking` | 660,011 | refused |
| `encrepo` | 383,384 | refused |
| `gpuml` | 299,127 | refused |
| `llmcollab` | 281,634 | refused |
| `selfescrow` | 166,012 | refused (barely) |
| `storage` | 154,233 | ok |
| `glucose` | 142,041 | ok |
| `dnnquant` | 123,942 | ok |
| `compression` | 50,466 | ok |
| `cryptotoken` | 48,414 | ok |

A million-token model does not help here: the refusal happens before any
provider call. Check the spec size first.

Resolving it means one of: implementing the retrieval-focused grounding step,
raising the threshold and accepting truncation (which the guard exists to
prevent), or documenting large specs as unsupported. Until then, roughly 160k
chars is the ceiling on what this tool can draft from.

## Context is the first filter among models that clear the threshold

A specification is roughly one token per four characters, and the whole spec goes
into the analysis and drafting calls. The tool sizes each Ollama request's
context window (`num_ctx`) to the prompt plus the output budget automatically, so
a model gets its full trained window with no per-model tuning and no context-baking
aliases; you only have to pick a model whose window is large enough. If a spec is
larger than the model's window, the draft path REFUSES up front (`checkSpecFits`,
the `SpecFocusThreshold` guard above and the per-model window check), rather than
silently drafting from a truncated spec.

Check the window before blaming the model:

```bash
ollama show <model> | grep "context length"
```

A 124k-character specification is ~31k tokens, which rules out every 32k-40k
model regardless of how well it drafts.

| model | context | takes a 124k-char spec? |
|---|---|---|
| `devstral-small-2:24b` | 393k | yes |
| `qwen3-coder:30b` | 262k | yes |
| `deepseek-r1:14b` | 131k | yes |
| `qwen3:32b` | 41k | **no** |
| `qwen3:8b` | 41k | **no** |
| `codestral:22b` | 33k | **no** |

This produces an awkward result worth stating plainly: `qwen3:32b` drafted the
best claims in the screen and **cannot be used on a full-length specification**.

## Measured against a real corpus (2026-07-30)

The screen above uses a 3.7 KB fixture. Ten real parent/continuation pairs were
then run end to end, specifications from 47 KB to 645 KB (median 154 KB, roughly
12k to 165k tokens). Two results matter more than any ranking.

**Coverage, not claim counts, is what separates local models.** Of the models
available locally, these drafted all ten specifications including the 645 KB one:

| model | drafted | resident memory | note |
|---|---|---|---|
| `gemma4:26b` | 10/10 | **20 GB, flat** | A4B mixture-of-experts |
| `gemma4:31b` | 9/10 | **30 GB, flat** | fails only the 645 KB spec |
| `qwen3-coder:30b` | 10/10 | 44 GB at 65k, **122 GB at 262k** | grows steeply with context |
| `gemma4:12b` | 9/10 | 12 GB, flat | uneven quality across specs |
| `devstral-small-2:24b` | 7/10 | 20 GB to **187 GB** | steep growth |
| `deepseek-r1:70b` | 6/10 | 111 GB at 65k | returns prose, not JSON, on several |

**Model file size does not predict memory in use.** `qwen3-coder:30b` is an 18 GB
download that occupies 122 GB once given a context window large enough for the
largest specifications. `deepseek-r1:70b` is a 42 GB download that occupies 111 GB.
The ratio ranges from about 1x to 6x and cannot be estimated from the download.

**Some architectures make long context nearly free, and that decides what fits.**
Both Gemma 4 variants hold a 645 KB specification in essentially the same memory as
a 47 KB one, because their attention design keeps the key-value cache almost
constant. Qwen3-Coder climbs 21 GB to 122 GB across the same range, and Devstral
20 GB to 187 GB. This is why a 31B model can fit a workstation while a 24B one does
not, and it is the single most useful thing to check before choosing a model for
long specifications.

## If a specification is refused, check `--max-tokens` before changing model

`--max-tokens` does two things: it caps the completion, and it reserves that much
of the context window. The reservation is what usually bites.

Measured across 127 drafts: the largest output was about 3,217 tokens against the
default 16,000 cap, the median about 1,292, and **no run was ever truncated**. The
cap never binds on output. It binds only by reserving context that nothing uses.

Consequence: several models that refused the 645 KB specification at the default
drafted it fine at `--max-tokens 8000`, which raises the usable spec ceiling by
roughly 8,000 tokens. They had missed by a few hundred. Before concluding a
specification is too large for a model, halve the completion budget and retry.

> **THIS DOES NOT APPLY TO A THINKING MODEL, AND FOLLOWING IT THERE MAKES THINGS
> WORSE. Corrected 2026-08-29.** The measurements above came from 127 drafts that
> contained no forced-reasoning models, so they are true of that sample and do not
> generalize. Reasoning tokens are billed against the SAME budget as the answer, so
> a model that deliberates until the budget is gone returns an empty completion
> with `finish_reason: length`. Measured on `z-ai/glm-5.3` against a 49 KB
> specification: **13,604 reasoning tokens and zero characters of output** at the
> 16,000 default. Halving to 8,000 truncates it sooner and still yields nothing.
>
> **Raising the budget does not fix it either**, which is the counter-intuitive
> part: at 40,000 the same call spent 41,944 tokens reasoning and still returned
> nothing, because the model reasons to fill whatever it is given.
>
> **The fix is `--reasoning-effort`, not `--max-tokens`.** See the next section.

This does not rescue a model whose window is genuinely too small. A 131k-token
window cannot hold a 165k-token specification at any output budget.

## If a model returns nothing after a long wait, it is a thinking model

Added 2026-08-29, from the batch C run.

Some models deliberate before they answer, and that deliberation is billed against the
completion budget. Given a budget they can exhaust, they will: they reason until it is
gone and return an empty result. The symptom is distinctive and easy to misread as the
model being slow or the tool being broken:

- a long wait, then nothing, with no error that names a cause
- more `--max-tokens` makes it slower and no more likely to succeed

**Set `--reasoning-effort`.** `low`, `medium` or `high`; unset keeps the provider's own
default, which for some models is effectively unbounded.

```
continuation-drafter draft --spec spec.txt --parent claims.txt \
  --model z-ai/glm-5.3 --reasoning-effort high
```

Measured on `z-ai/glm-5.3`, same 49 KB specification, same 16,000-token budget:

| | result |
|---|---|
| provider default | 13,604 reasoning tokens, **0 characters**, no draft |
| `--reasoning-effort high` | a complete 16-claim draft in 13 seconds |

**`high` is the right starting point.** It is the least restrictive setting that still
guarantees the model emits an answer, and it costs nothing in quality: measured across
ten specifications in one panel, `gpt-5.5` scored 88.8 at `high` against 88.6 at its
default, and `glm-5.2` 85.5 against 84.4. Both differences are inside this benchmark's
~3-point floor. **Reasoning effort rescues models that would otherwise produce nothing;
it does not make a working model draft better.**

Models measured to need it: `glm-5.3`, `glm-5.3-flash`, `kimi-k3`, `kimi-k2.6`,
`nemotron-3-super`, and the `qwen3.8` family. Models that were unaffected either way:
`gpt-5.5`, `glm-5.2`, `llama-4-maverick`, `deepseek-v4-flash`.

## What the screen actually established

**Coder-tuned models draft fine.** An earlier experiment found `codestral:22b`
fabricating 100% of its spec anchors and the conclusion drawn was that coder
models were the wrong class for patent work. That conclusion was wrong.
Anchor extraction is a retrieval task; drafting is generative prose under formal
constraints, and `qwen3-coder:30b` scored the highest rate of proper
transitional phrases in the set. Judge a model on the task you are giving it.

**`codestral:22b` is genuinely weak here**, but for a specific reason: it emits
`- ` bullet hyphens inside claim bodies. That is a form defect, not a style
preference.

**`glm-4.7-flash` cannot be used**, despite having the largest context of any
general-purpose model available. It returns structurally broken JSON even on a
trivial request, with thinking already disabled:

```json
{"targets": [{"scope": "A Valve", "\"system\": \"valve\""}, "{\"description\":...
```

That is a model capability limit, not a configuration problem.

**`deepseek-r1:14b` produced zero structured claim bodies.** Its claims are
prose-shaped rather than element-delimited.

## Thinking models

`qwen3`, `glm`, and `deepseek-r1` are thinking models. On Ollama's native
`/api/chat` their reasoning returns in a separate field but **still counts
against `num_predict`**, so the model can spend its whole budget reasoning and
return a truncated, empty answer.

The binary sends `think: false` by default, so this is handled. Set
`CD_OLLAMA_THINK=1` to restore the model's own default when comparing behavior.

Symptom if you ever hit it: `done_reason=length` with no usable content, on an
input far too small to justify it.

## Remote models (OpenRouter)

Any model with a **slash**, or a recognized remote prefix (`claude-`, `gemini-`,
`grok-`, `gpt-`, `chatgpt`, or an o-series name like `o3`), routes to OpenRouter
instead of local Ollama. A bare recognized name is rewritten: `claude-opus-4-8`
becomes `anthropic/claude-opus-4-8`, `gpt-5.5` becomes `openai/gpt-5.5`.

```bash
export OPENROUTER_API_KEY=...
continuation-drafter draft --spec s.txt --parent p.txt --model anthropic/claude-opus-4-8
```

**Since 2026-08-25 there are three cloud providers, and `--provider` chooses between them.**
The rewrite above is OpenRouter's own addressing scheme, so it applies to OpenRouter alone:
name Anthropic or OpenAI directly and the model goes as you wrote it.

| Provider | `--provider` | Key | Endpoint override |
|---|---|---|---|
| OpenRouter | `openrouter` | `OPENROUTER_API_KEY` | `OPENROUTER_BASE_URL` |
| OpenAI | `openai` | `OPENAI_API_KEY` | `CD_OPENAI_BASE_URL` |
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` | `CD_ANTHROPIC_BASE_URL` |

```bash
export ANTHROPIC_API_KEY=...
continuation-drafter draft --spec s.txt --parent p.txt \
  --provider anthropic --model claude-opus-4-8
```

### What the picker lists, and what it leaves out

The model picker curates each provider's catalog rather than showing it whole. Three of the
rules are about capability and one is an editorial choice, which is stated because they are
not the same kind of decision:

| Rule | Why |
|---|---|
| Text output only | read from the provider's own capability field, not guessed from the name |
| No purpose-built variants | `codex`, `-search`, `deep-research` and `instruct` builds take a prompt but are specialised away from drafting |
| No suffixed variants | `:free`, `:batch`, `preview` and `-exp` are the same model under different billing or stability terms |
| Nothing older than about 18 months | keeps the current and previous generation; the rest are not serious candidates |
| **Major labs only, on OpenRouter** | **an editorial choice, not a measurement.** See below |

**"Most used" is not something this tool can know.** OpenRouter's API publishes no usage or
ranking data: `/models` returns the same entries in the same order whether or not an
`order=top-weekly` parameter is supplied, and there is no rankings endpoint. So its catalog is
narrowed by a list of frontier labs, which is a judgement about which models a practitioner
would draft with, not a popularity ranking. It will go out of date as labs come and go.

**Nothing is unreachable because of any of this.** With an empty search box the picker renders
the fifty most recent and says how many more there are; typing filters the whole curated set;
and every provider pane has a box to name a model directly, which reaches past the catalog
entirely. That last one matters for Anthropic, whose OpenAI-compatible endpoint serves chat but
answers `/v1/models` with 401, so its catalog is fetched over the native API instead.

The flag is needed because the model string cannot express the choice: `anthropic/claude-opus-4-8`
is OpenRouter's slug for that model, so the same string would name two different destinations.
**With no `--provider` the string still decides, exactly as described above**, which is what
keeps every command in this document working unchanged.

**Ambiguous bare names default to local, by design.** Families that exist BOTH as
local Ollama models and as remote models (`llama`, `qwen`, `deepseek`, `mistral`)
are deliberately NOT auto-routed: a bare `deepseek-chat` is treated as a local
Ollama tag, because the string alone cannot tell a legitimate local `llama3.1`
from a remote model the user forgot to slash, so the safe default is local. To
force remote, write the `vendor/name` slug (e.g. `deepseek/deepseek-chat`). If a
bare name is not a pulled local model, the run fails with a not-found error that
now hints at the remote form.

**Kimi: use `kimi-k2.5`, not `kimi-k3`.** Both are reasoning models. `kimi-k3`
(1M context, $3/$15 per M) spends a large reasoning budget and drafts a single
pair in ~33 minutes, 6x the fastest model, and timed out once on the corpus.
`kimi-k2.5` (262k context, $0.57/$2.85) is the smaller, faster variant: ~9
minutes on the same pair, 1/5 the cost, comparable output. 262k context clears
everything under the 160k-char draft threshold anyway. The base `kimi-k2` is
faster still (non-reasoning) but its OpenRouter provider (Novita) does not
support structured-output mode, which this pipeline requires, so it fails at the
target-extraction step.

**Confidentiality.** A remote model means the specification leaves the machine
and reaches a third party. For published patents that is fine. For an
unpublished or privileged application it is not: use a local model. The
"nothing leaves your machine" property belongs to the local path only, and is
not a property of the tool as a whole. Two things can leave, and they are different in kind:
your SPECIFICATION, only ever to a model provider you configure, and, if you hold a paid
licence, your LICENCE KEY to the renewal endpoint about once a month. The second carries no
specification text and is disabled by `CD_NO_LICENSE_RENEWAL=1`.

**The local path costs more than speed and context.** On a real spec, the strong
local models drafted well-grounded, cleanly-formed claims that were aimed at the
single most literal locus in the disclosure, while frontier models spread their
independent claims across the strategically relevant loci (and reproduced that
aim run-to-run). So the local-vs-remote choice is not only privacy-for-speed, it
is privacy-for-strategic-reach: a local draft can be perfectly supported and
still strategically flat. For strategy-sensitive drafting on material that may be
sent remotely, prefer a frontier model; keep the local path for anything
privileged and treat its aim as a first pass a practitioner sharpens.
