# Changelog

All notable changes to continuation-drafter are recorded here. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and releases
are the `v*` tags in the source repository.

This file is the **only** source of release notes: the release pipeline
deliberately does not generate them from commit history, because the source
repository is private and its commit subjects are not published.

## [0.1.0] - 2026-08-05

**First release.** The binary is published to
[`patent-continuation-public`](https://github.com/Obviously-Not/patent-continuation-public);
the source stays private. All inference runs locally by default, so nothing
leaves your machine unless you point the tool at a remote endpoint yourself.

A `0.x` version on purpose. This is R&D-grade drafting input for a licensed
practitioner, not a finished product, and the limitations below are the reason
rather than boilerplate. Read them before using output for anything real.

### Added

- Go implementation of the full drafting pipeline: `draft` (spec analysis,
  target extraction, drafting, and a verify-and-revise loop), `critique`
  (cross-family critic that flags per-claim defects), `judge` (claim-quality
  rubric with spec-anchor validation), `panel` (multi-model scoring), plus
  `models`, `version`, `license`, `activate`, and `serve`.
- A local web UI on `localhost:9473`, served by the binary itself with no build
  step. HTML and CSS are embedded in the binary.
- A provider layer covering local Ollama and any OpenAI-compatible remote
  endpoint. Keys resolve from the environment or a mode-0600 config file, never
  from a flag and never from a tracked file.
- Offline Ed25519 license verification supporting multiple keys and capability
  tiers.

### Known limitations

- **R&D-grade, not filing-ready.** The pipeline drafts real, well-formed,
  spec-grounded continuation claims, but a single AI judge cannot reliably rank
  drafters. Only the multi-model panel is stable, and even it separates gross
  differences rather than near-equals. Treat every output as drafting input for
  a licensed practitioner exercising independent professional judgment, never as
  a legal determination.
- **License tiers are reported, not enforced.** Verification and reporting are
  implemented; feature gating is deferred by owner decision, so all capabilities
  are available regardless of tier. Release builds also carry no embedded public
  key yet, which is a build-time gap rather than a policy change.
- **No coverage metric.** The embedding-based coverage map from the originating
  experiment was not ported. This is a named, accepted gap.
- Model choice matters more than usual, because a specification must fit inside
  the model's context window. Read [docs/models.md](docs/models.md) before
  choosing one for real work.
