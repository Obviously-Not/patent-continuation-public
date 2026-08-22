# Changelog

All notable changes to continuation-drafter are recorded here. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and releases
are the `v*` tags in the source repository.

This file is the **only** source of release notes: the release pipeline
deliberately does not generate them from commit history, because the source
repository is private and its commit subjects are not published.

## [Unreleased]

Nothing yet.

## [0.1.5] - 2026-08-21

### Added

- **A macOS app you open by double-clicking it.** Download
  `continuation-drafter-macos.dmg`, drag **Continuation Drafter** to Applications, and
  double-click it: the drafting interface opens in your browser. No Terminal, no
  permissions to change, and one download that runs on both Apple silicon and Intel.

  This exists because the previous release could not be opened that way and never could
  have. A binary downloaded through a browser arrives without an execute bit, and a bare
  file has no bundle, so double-clicking it opened a text editor showing the program's
  raw bytes. Signing fixed a different problem (macOS killing the file silently) and left
  this one untouched.

  The app is signed with a Developer ID certificate, notarized by Apple and **stapled**,
  so first launch needs no network check. macOS still asks once whether you are sure you
  want to open something downloaded from the internet; the button is **Open**.

  It has no Dock icon, because it runs no window of its own. Press **Stop** in the
  browser tab to shut it down.

- **A Stop control in the web interface.** The only way to end a session started from the
  app: there is no terminal to interrupt and no Dock icon to quit.

### Changed

- **The command-line binaries are unchanged and still published.** They remain the right
  download for a terminal, a script or CI, and the install steps for them are the same.
- **`CD_PORT` now applies to a bare invocation**, not only to `serve`. It set the port for
  one of the two ways of starting the interface and silently did nothing for the other.

### Fixed

- **Starting the interface twice no longer fails to bind.** A second double-click, or a
  second `continuation-drafter` in another terminal, now opens the browser at the session
  already running instead of reporting that the port is in use.

## [0.1.4] - 2026-08-21

### Added

- **The macOS binary is now signed and notarized by Apple.** Downloading it no longer
  produces the "Apple could not verify" dialog, whose default button is *Move to
  Trash*. Nothing has to be done to the file after downloading: no `chmod`, no
  clearing of the quarantine flag, no Terminal at all if you do not want one.
- **Running `continuation-drafter` with no arguments opens the drafting interface in
  your browser.** Drafting, per-claim critique, scoring and the multi-model panel are
  all there, and the models it offers are the ones already installed on your machine.
  The command line is unchanged and `--help` still lists every command; a bare
  invocation from a script or CI still prints usage and exits rather than starting a
  server.

### Changed

- **`models` now recommends a model you already have.** It reported the installed
  count and then suggested downloading something else, which on a machine with a
  full Ollama library meant being told to fetch tens of gigabytes for no reason.
- **The command list is grouped**, with the interface, model check and drafting under
  "Start here" instead of twelve commands in alphabetical order.

### Fixed

- A next-step loop, and duplicate steps, in the guidance printed after a command.
- `upgrade` advertised a purchase in its help text while correctly refusing to make
  one, since there is nothing to buy.

## [0.1.1] - 2026-08-18

### Security

- **The local web UI now refuses cross-origin and non-loopback requests.** Any web
  page you had open in another browser tab could previously POST to the tool's
  local server. The damaging case was the Ollama host setting: a hostile page
  could point it at a machine it controlled, and every later run of a *local*
  model would then send your specification there while the tool still reported it
  as local. Requests must now come from a loopback origin and be addressed to a
  loopback host, which also closes the DNS-rebinding variant. If you reach the UI
  through a hostname or a proxy, use `http://127.0.0.1` directly.
- **The configuration file's permissions are now repaired on every save, not just
  when it is created.** A config that had become world-readable by any route
  stayed that way, with your API key in it. Saves are also atomic now: an
  interrupted write can no longer destroy your license keys and API key together.
  On Unix the file is always restored to owner-only.

### Added

- **`draft --demo`, a complete run that needs nothing from you.** The download is a
  single binary, so before this you had to supply a specification and its filed
  claims before the tool would do anything at all. The walkthrough specification
  the published lessons use is now built in: `continuation-drafter draft --demo`
  drafts against it from any directory. It never counts against any allowance.
- **`models` now tells you what this machine can run**, instead of only what you
  have configured. It reports how much memory you have, whether Ollama is
  reachable, and which measured model fits with room to spare, saying whether you
  already have it or need to pull it. Where memory cannot be detected it says so
  rather than guessing, and it states plainly that the best local model still
  scores materially below a remote one.

- **`export`, which writes your drafted claims to a Word document.** Claims come
  out of this tool as text, and the work continues in a word processor, so
  `export --draft claims.txt --out claims.docx` gives you a `.docx` that opens in
  Word, Pages or LibreOffice. It also reads a `draft --json` run straight off a
  pipe, so drafting and exporting are one command. **Your claim numbering is
  preserved exactly as drafted and never renumbered**, since a continuation's
  claims are commonly numbered on from the parent. The file is written owner-only
  (mode 0600) because it holds client claim language, and it will not overwrite an
  existing file unless you pass `--force`, so re-running the command cannot destroy
  edits you made in Word. The maturity notice is written inside the document, where
  it is visible to whoever opens the file later. Free, and not licence-gated.
- **LaTeX export**, for drafters who keep matter documents that way:
  `export --draft claims.txt --out claims.tex`. The format follows the output
  extension, so you do not have to say it twice; pass `--format` to override.
- **Licence renewal, for paid subscriptions only.** So that you activate a licence
  once and never handle a key again, the binary fetches a replacement when the
  current one is close to expiring: roughly once a month, on launch, not on a
  timer. It sends the licence key and nothing else, and carries no specification
  text, no matter identifiers and no usage data. Set `CD_NO_LICENSE_RENEWAL=1` to
  switch it off entirely; a licence that is never renewed simply works until it
  expires, and the tool keeps working normally if the renewal cannot be reached.
  **Without a paid licence the binary makes no such call at all.**
- **A dependency and standard-library vulnerability scan now runs on every change, and
  again when a release is built.** The second one is the half that reaches you: the
  release pipeline resolves its Go toolchain fresh when it runs, so the standard library
  compiled into a published binary is not necessarily the one that was scanned when the
  change was reviewed. The scan runs before anything is published, so a release stops
  rather than shipping a binary built from an unscanned standard library.
- A warning if a build's embedded licence key is malformed, so a broken build says
  so instead of quietly behaving as unlicensed.
- **A `See also` line at the end of the suggestions**, pointing at the
  documentation site, so a pointer to something worth reading does not have to
  pretend to be a command you can run.
- **Every command now tells you what to do next, in prose as well as in `--json`.**
  Previously the suggestions existed only under `--json`, so running the tool
  normally showed none at all, and `version` showed none either way. Each
  suggestion now carries a reason, a priority, and where one exists a command you
  can copy and run. **Failures carry them too**: if a run fails because Ollama is
  not running, or a key is missing, or the output was cut off, the error now comes
  with the specific thing to try rather than only the message.

### Fixed

- **The local web interface now tells you what to do when a drafting run fails.**
  A run that failed because a model was missing or the provider was unreachable
  showed the raw error and nothing else, while the same failure at the command
  line printed a full set of recovery steps. Progress and results arrive over a
  stream rather than as an ordinary reply, and the recovery steps were only ever
  attached to ordinary replies. The steps a finished run suggests are now shown
  too; before, the panel kept showing the advice from the moment the run started.
- **A mistyped flag or command now points you at the flag list instead of
  suggesting a drafting run.** `draft --xyz`, an unknown command, a bad numeric
  value and a missing required flag all fell through to generic advice whose
  first suggestion was to run the tool against its own test fixture.
- **Running without choosing a model now tells you to pick a model.** The
  first error most new installs hit ("no drafting model specified") was answered
  with "Start Ollama, or point at a running one", because the error's own help
  text mentions Ollama by name. Starting a server does not choose a model; the
  advice now lists what is reachable and says how to set a default.
- **A web page reconnecting to a finished run is no longer told to change
  models.** Results are held for a few minutes after a run completes; a page
  that came back later saw "job not found" answered with model advice. It now
  says the run has expired and to start it again.
- **A model the provider does not have is no longer reported as the provider
  being down.** Asking for a model that is not pulled locally suggested starting
  Ollama, which was already running. It now suggests listing what is reachable
  and pulling the model.
- **Suggested commands now work on a downloaded copy of the tool.** Several
  suggestions named `open`, which exists only on macOS, or pointed at files that
  are only present if you built the tool from source. The download is a single
  binary and carries neither. Suggestions that name a document now give its
  address, and the ones that use the bundled example appear only when that
  example is actually present.
- **A drafted claim no longer ends in a stray `---`.** Models separate groups of
  claims with a horizontal rule, and the rule was being kept as part of the claim
  before it. It reached the exported Word document, and the critique and scoring
  passes read it as if it were claim language, so an unpredictable handful of
  claims in every run needed the same manual deletion.
- **The local web interface no longer preselects a model your machine cannot
  run.** It picked whichever installed model had the most parameters, with no
  reference to how much memory you have, so on a machine holding a large model
  library the first click could only fail. It now prefers the strongest model
  whose weights fit in memory. Where memory cannot be detected it does not filter
  at all, which is the same honest-unknown rule `models` follows.

### Changed

- **BREAKING for `--json` consumers: `next_steps` is now an array of objects, not
  an array of strings.** Each entry is
  `{action, command?, priority, reason, timing?}`. If you parse `next_steps` as
  strings, read `.action` instead. Nothing else in the envelope changed, and
  `data` and `notice` are untouched.
- **`upgrade` now tells you there is nothing to upgrade to, instead of opening a broken
  page.** The command opened a payment link that returned an access-denied error, and
  always would have. There is no paid tier to buy yet; every feature is available in the
  build you have, and no licence key changes that. The same applies to the Upgrade button
  in the local web interface, which now says the link is not configured rather than
  opening the error page.
- **The startup banner no longer names a scoring criterion that does not exist.**
  It read "scores claim drafting quality (form, support, definiteness, craft)".
  There is no `form` criterion, and the list also left out one of the criteria
  that can veto a claim outright. It now says what the tool scores against rather
  than naming a partial and partly invented list.

- **Licence tiers collapsed to a single `pro` tier.** The earlier set (`panel`,
  `export`, `batch`, `history`, and the domain packs) was a guess at which axis to
  price before any of it had been sold. Notably, `panel` is gone: the multi-model
  panel remains free, because this project's own testing shows a panel score
  separates only gross differences and cannot rank near-equal drafts, and charging
  for it would imply otherwise. Drafting and critique remain free.
- **Corrected how this documentation describes API-key handling.** It said keys
  came from the environment only. In fact the key is read from the environment
  first and otherwise from `~/.continuation-drafter/config.json`, which the binary
  creates owner-readable and which the web UI can write for you. The environment
  always wins. Never a command-line flag and never a log.

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
