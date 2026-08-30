# Changelog

All notable changes to continuation-drafter are recorded here. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and releases
are the `v*` tags in the source repository.

This file is the **only** source of release notes: the release pipeline
deliberately does not generate them from commit history, because the source
repository is private and its commit subjects are not published.

## [Unreleased]

## [0.2.4] - 2026-08-29

### Added

- **A bill of materials with every download.** Each binary now ships an SBOM listing every
  library inside it, with versions, in the standard SPDX format. It detects nothing by
  itself; it is what lets you answer "does the build I am running contain this" months
  from now, when an advisory lands against a library, without rebuilding anything. The
  documents are covered by `checksums.txt` like every other file.

### Changed

- **The download page now leads with your own operating system.** The button, the
  instructions, the command examples and the order of the binary list all follow the
  machine you are reading on. A Windows visitor was previously offered a macOS `.dmg` as
  the only button on the page, with commands underneath that their shell could not run.

- **A page explaining what is checked, and how you can check it.** Where your
  specification goes, who signed the download, the published SHA-256, and the
  vulnerability scan that gates a release. All of it was already true and all of it was
  buried inside the install steps.

### Notes

- **Nothing about drafting changed.** The pipeline, the models, the skills and the local
  web UI are identical to 0.2.3.

## [0.2.3] - 2026-08-28

### Changed

- **The Windows binary is signed.** Windows Defender and SmartScreen blocked the previous
  download, because the `.exe` carried no signature at all: the macOS builds were signed and
  notarized and Windows had nothing. Releases are now signed with an Azure Artifact Signing
  certificate issued to Geeks in the Woods, which is the name Windows shows as the verified
  publisher. The signature is timestamped, so it keeps verifying after the signing
  certificate itself expires.

  The release refuses to publish an unsigned Windows binary rather than shipping one quietly.
  It asks Windows itself whether the signature verifies, and requires both a valid status and
  a timestamp before the release leaves draft. A signing step that silently matched nothing
  is how an unsigned macOS binary shipped in v0.1.3.

- **`checksums.txt` changes for the Windows entry.** Signing rewrites the file, so its hash is
  not the one an unsigned build would produce. Verify a download against the checksums
  published with the same release, never against an earlier one.

### Notes

- **0.2.2 was tagged and published nothing.** Its Windows signing step failed on a path bug,
  and the pipeline did what it is built to do: the release stayed a draft rather than
  publishing a binary that was unsigned or whose checksum did not describe it. There is no
  0.2.2 download and there never was one.

- **Nothing about drafting changed in this release.** The pipeline, the models, the skills and
  the local web UI are identical to 0.2.1. If Windows was not blocking your download, there is
  nothing here you need.

## [0.2.1] - 2026-08-26

### Added

- **Claims are laid out one element per line.** A claim sets out its elements separated by
  semicolons, and the page was showing all of them as a single paragraph: one claim measured
  3,284 characters as an unbroken block. Each element now begins its own line, indented, which
  is the form the rules prescribe for a filed claim and the form a practitioner reads. The
  wording is untouched; only the whitespace differs. Word, PDF and the plain-text export carry
  the same layout, and so does Copy.

- **You can check claims without rewriting them.** The only thing you could do with a drafted
  set was ask a model to revise it, which changes your claims. There is now a check that
  reports what is wrong and alters nothing, offered first on claims nothing has reviewed and
  alongside revising when findings exist.

- **A list of your claims, beside them.** Each row shows the claim number, whether it is
  independent or which claim it depends on, what distinguishes it, and how many findings it
  carries. Clicking one jumps to it.

- **Answers in the Discuss tab are formatted.** Headings, lists, bold and quoted terms used to
  arrive as their own punctuation.

### Changed

- **A revision shows only the elements that changed.** A rewrite touching two elements of ten
  was rendered as the entire claim in red and green; the untouched elements now sit quietly
  and the changed ones stand out.

- **Claims and answers use more of the window.** Both were held to the width of a paragraph of
  prose, which suits an essay and not a claim.

- **The page says what to do next in every state**, including when a claim set has been drafted
  but not reviewed, and it always offers a way to open the conversation about it.

### Fixed

- **A drafting run's work reaches the matter it belongs to.** Claims, findings, craft flags and
  continuation targets were computed and then written nowhere, so the Discuss tab could see
  none of it and the monthly counter could not tell a re-run from a new matter.

- **A file dropped onto the page is saved.** Choosing one with the file picker saved it;
  dropping the identical file did not, which was invisible until the Discuss tab reported
  having no application.

- **A review that fails says so.** When the critique pass could not complete, the page showed
  no findings and no explanation, and reopening the matter then reported the claims as
  reviewed and clean. Whether the check actually ran is now recorded and shown.

- **A malformed reply from the model is asked again.** Roughly one review in eight came back
  with a stray character that made it unreadable, and the whole review was discarded for it.
  The retry also varies its sampling, since repeating an identical request produces an
  identical answer.

- **Errors say what happened and keep the evidence.** A failed review reported only a byte
  count. It now distinguishes a reply that ran out of room from one that was never valid, and
  quotes what came back.

- **A reference to a section of your specification is no longer mistaken for a citation of
  law**, and a reference to the parent's claims is no longer reported as invented.

- **Asking how to improve a claim gets claim language**, not a refusal about filing strategy.

## [0.2.0] - 2026-08-25

### Added

- **OpenAI and Anthropic join OpenRouter as cloud providers.** Choose with `--provider
  openai|anthropic|openrouter`, or pick a provider tab in the model picker. Each reads its own
  key (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY`) from the environment or
  from the same private config file, and the web interface can save one for you. Nothing about
  where keys come from has changed: never a flag, never a log, never the repository.

  **Every command you already use keeps working.** With no `--provider`, the model string
  still decides exactly as before: a colon tag runs on this machine, a slash slug or a bare
  vendor prefix goes to OpenRouter. The flag exists because the model string cannot express
  the new choice: `anthropic/claude-opus-4` is OpenRouter's own name for that model, so the
  same text would name two different destinations.

- **The privacy note names the provider and the endpoint.** It said "the remote provider",
  which was as specific as it could be when there was one. With three, the single fact it
  exists to convey is which company receives your specification, so it now says so, in the
  terminal and in the browser.

- **The model picker lists each provider separately and curates what it shows.** Text models
  only, no purpose-built variants (codex, search, deep-research), no billing or preview
  variants, and nothing much older than the previous generation. OpenRouter is further limited
  to the frontier labs, which is an editorial judgement rather than a popularity ranking:
  no usage data is published, so no such ranking can be computed. With an empty search box you
  see the fifty most recent and a count of the rest; searching reaches the whole curated list.
  Every provider pane also has a box to name a model directly, which reaches past the catalog.

- **Discuss can look for broader scope in claims you already have.** Ask how to widen a
  claim and the tool works through four moves against your existing claims: who performs the
  reciprocal or downstream operation, which limitations describe one implementation rather
  than the invention, whether a per-element step can be claimed as an aggregate, and whether
  the structure the process produces can be claimed instead of the process. Until now the
  only mapping pass looked away from your claims, at what else the specification disclosed,
  so a question about the claims themselves had nowhere to go.

- **Discuss can report which limitations are narrowing you for nothing.** Ask what a claim
  set gives away and the tool reads each limitation against the specification and reports the
  ones the disclosure does not require: a specific term where a broader category is
  disclosed, a step tied to one component that could happen elsewhere, a recited mechanism
  where only the result matters, or an "each" the disclosure says can be a portion. Each
  finding names the sentence of your specification that supports the broader alternative and
  says whether to widen the claim or keep the narrow position as a dependent claim. Available
  in the conversation and at `POST /api/limitations`.

- **Discuss can describe what keeping your current claim center would look like.** Not every
  continuation goes somewhere new. Ask about filing another application on the same invention
  and the tool states the inventive center as a workflow, identifies whether the entity you
  nominally claim (a server, a backend, a service, a client) is essential or merely what
  happens to execute it, points out recitations that only establish that software runs on a
  computer, and names which existing dependent claims are worth carrying forward. It never
  recommends preserving: whether your current center deserves another family member turns on
  commercial and prosecution judgement it cannot see.

- **The broadening pass considers two more moves.** Alongside actor, abstraction, granularity
  and artifact, it now asks which phase of the system's life is unclaimed (provisioning,
  configuration, update, migration, expansion, recovery, retirement) and which other product
  disclosed in your specification could perform the same role, since a role can stay identical
  while the product performing it changes.

- **The continuation directions found for you are listed beside the conversation**, each
  labelled with whether its supporting quote was located verbatim in your specification, and
  each removable. Directions you keep are what the drafting pass writes claims to.

### Changed

- **What you write in a message now reaches the pass that runs.** Previously every pass ran
  on the document and the tool's own analysis, and your wording was discarded before the
  model saw anything, so a paragraph of drafting instructions changed nothing. It now steers
  the pass it triggered.

- **The assistant no longer suggests what to do next.** It was recommending capabilities of
  this product without ever being given the list of them, so the names were invented. The
  next steps beside your reply are derived from what your session actually contains, and are
  now the only place that advice comes from.

- **Drafting asks for a second statutory class where your specification supports one.** Of
  nine real continuations examined, six pair a method claim with a device, system or
  computer-readable-medium claim, and the drafting prompt had never raised the question. A
  method claim reaches the party performing the steps and a device claim reaches the party
  making or selling the thing, so a family in one class leaves the other party unreached. It
  is conditional: a method the specification never embodies in a described device does not
  earn a device claim, and padding a class with a restatement of the first claim is forbidden.

### Fixed

- **A claim set always comes with dependent claims.** Drafting from a single direction
  produced one bare independent claim and no fallback positions, because the instruction
  asked for a set of three or four independents and the model read it as inapplicable. A
  family without dependent claims is missing the claims that matter when the independent one
  is rejected. Drafting from one direction now yields that claim plus two to four dependents,
  and a set that comes back without any says so.

- **A large specification no longer fails with an error about the connection.** On
  specifications over roughly 150 KB the local model could stop mid-answer after producing a
  usable result, and the tool discarded it and retried four more times before reporting a
  transport error. The usable part is now kept, and a stop that carries an answer is no
  longer treated as a temporary glitch worth repeating.

- **Describing a problem in a claim is enough to have it fixed.** Asking to rewrite a claim
  you had just described a defect in was refused with a request for a critique first, even
  though the rewrite never used one.

- **The first document attached to a matter is no longer announced as a replacement.** It read
  "Replaced the document with ..." and drew a divider saying everything above it described a
  document no longer loaded, above a transcript with nothing in it. A matter has an identity
  from the moment you create it, and that was being read as a conversation already holding
  something.

- **Reopening a matter shows what it is working from.** The row naming the loaded application
  stayed hidden on a matter opened from the sidebar, and attaching a second document to one
  discarded the earlier exchange while the transcript stayed on screen.

- **A run that times out twice now stops instead of timing out five times.** A timeout can be
  a busy machine, and one retry still covers that. It can equally be a property of the request,
  and then every further attempt re-sends the same document and waits the same deadline for the
  same answer. On a 660 KB specification that cost fifty minutes locally to learn what the first
  attempt had already established, and would have cost longer on a remote provider.

- **A timeout no longer tells you to start a model server that is already running.** The advice
  now leads with the size of the specification, which is what a timeout on a retry usually
  means: a document can fit a model's context window and still take longer than any deadline.
  Timeouts from remote providers previously produced no specific advice at all.

- **Advice about remote models names every provider.** Two messages still said a remote model
  runs on "your own OpenRouter key" after OpenAI and Anthropic were added.

- **A model on your own machine no longer asks for a cloud key.** Selecting a local model
  while a cloud provider was saved as the default produced "model \"gemma4:26b\" on provider
  \"Anthropic\" needs ANTHROPIC_API_KEY", which is a fair question to be confused by: it did
  not need one. A model name carrying a colon is Ollama's own naming and says where it runs,
  so a saved default no longer overrules it. **With a cloud key already saved this was worse
  than an error**, because the specification would have been sent to that provider while the
  model chosen was sitting on the local disk. Naming a cloud provider explicitly still works
  and now says plainly that the provider does not use that kind of name.

- **The model picker and the tool agree about where a pass will run.** The provider chosen in
  the picker was saved in the browser and never read back, so after a reload a remote model
  was tagged with a provider name that no longer existed, and a computed default told the
  browser one thing and the tool another. Both now record the same choice, and a provider name
  the tool does not recognise is refused when it is set rather than surfacing later as a
  failed drafting run.

- **A revision you keep is now actually kept.** Revising produced a proposal, and the button
  that adopted it changed only the page in front of you. Reloading brought the previous claims
  back, and the Discuss tab, which works from the matter, went on answering about the claims
  you believed you had replaced, with nothing saying so. Keeping a revision now saves it, and
  Undo saves the originals back, so the conversation and the page never disagree about which
  set is current. The button also names what it does: it said "Put these in the claims box",
  and that box had been removed some time ago.

- **The draft page says what to do next in every state it can be in.** One notice did the work
  of three. After a revision it still read "Next step: act on the findings below" above the
  proposal it had just produced, and pressing that button again would have started over from
  the original claims and discarded the proposal. When nothing was flagged it said nothing at
  all, so a clean set of claims simply ended the page. There are now three: revise the
  findings, go and read the revision that is waiting, or, when nothing is flagged, ask about
  the claims or export them.

- **A dropped file is saved to the matter, the same as a chosen one.** Attaching a document by
  clicking Browse saved it; dropping the identical file on the same box did not. Drafting
  worked either way, because the page sends its own copy of the text, so the difference was
  invisible until the Discuss tab, which works from the matter, answered "that needs
  specification, which this session does not have" about a document plainly on screen.
  Dropping a file also names an untitled matter now, which only the Browse path did.

- **A drafting run's work reaches the matter it belongs to.** The page never told the run which
  matter it was for, so everything a Draft run computed, the claims, the findings, the craft
  flags and the continuation targets, was written to nothing. The Discuss tab could see none
  of it, and the monthly counter could not tell a re-run of a matter from a new one.

- **The continuation targets a drafting run identifies are kept and shown.** The run reported
  "5 drafting target(s) identified" in its own progress log and the panel beside it was empty,
  because the targets were used to draft and then discarded. They now appear in the rail, with
  the same in-spec or strategy badge as everywhere else. A run fills an empty rail and never
  replaces targets you have edited, since those are what the conversation drafts from.

- **A reference to a section of your specification is no longer mistaken for a citation of
  law.** The tool replaces any answer that cites a statute, because an unchecked citation is
  the least reliable thing it could show you. It was reading "supported by Section 10" and
  "Sections 14 and 20 of the application" as citations, so a full claim-by-claim analysis was
  thrown away and you were told the tool had tried to quote law when it had not. Real
  citations are still caught, including invented ones.

- **A reference to the parent's claims is no longer reported as invented.** The tool warns when
  an answer mentions a claim number that does not exist. It counted only the drafted set, so
  with ten drafted claims and a parent numbered to twenty, "drafted claim 8 captures elements
  from the source claims 14 and 20" was flagged as unreliable. That comparison is the tool's
  purpose, and the warning was discrediting correct work.

- **Asking how to improve a claim gets claim language, not a refusal.** "How can I improve
  them" was answered with "I cannot advise you on whether to file certain claims", because the
  rule against advising on filing was being read as covering how to word a claim. A filing
  decision is whether to pursue a claim and remains yours; how to word one is drafting, and
  drafting is what this tool is for. It still does not assess patentability, novelty,
  non-obviousness, eligibility or infringement, and still does not quote law.

- **PDFs printed from a browser can be read.** A patent application saved as a PDF from
  Chrome or any Chromium-based browser was refused as unreadable, with a message suggesting it
  was a scan. It was not: the text was there and other readers extracted it cleanly. Five
  separate faults in the PDF reader had to be corrected together, since each one alone left
  nothing usable. Hex-encoded text, which is what a browser writes, was skipped entirely.
  Character ranges in a font's character map were ignored, and a range is how a font describes
  an alphabet. Every font in the document was read through one shared table, so subset codes
  from one font were decoded as another font's letters. Images were parsed as though they were
  text. And each line was broken wherever the writer moved the cursor, which this kind of file
  does for every single character.

- **Ligatures no longer break the check that a quotation is real.** Text extracted from a PDF
  carries typographic ligatures, so "satisfies" arrives as a word containing one combined
  character. The supporting passage recorded for a target is verified by matching it against
  the application word for word, and a passage written with ordinary letters could not be found
  in text spelled with ligatures, so a target with a perfectly good quotation was reported as
  having none. Extraction now folds them to ordinary letters, as other PDF readers do.

## [0.1.7] - 2026-08-21

### Fixed

- **The browser tab icon is the current logo again.** The local interface shipped an
  earlier mark on an opaque white plate while the website carried the not-equal mark, so
  the tool you ran looked like a different product from the one that described it, in the
  tab you leave open for a whole drafting session. It is now the same mark, transparent,
  and 1 KB instead of 70 KB.

## [0.1.6] - 2026-08-21

### Added

- **The macOS app has an icon.** The ObviouslyNot not-equal mark, in the brand gradient,
  on a rounded plate. It shipped in 0.1.5 with the generic application icon, which on the
  artifact whose whole job is to look trustworthy at first contact is the first thing a
  practitioner sees.

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
  are available regardless of tier. Release builds DO carry the licence public
  key, since v0.1.1 on 2026-08-18, so the deferral is a policy choice and no longer
  also a build-time gap.
- **No coverage metric.** The embedding-based coverage map from the originating
  experiment was not ported. This is a named, accepted gap.
- Model choice matters more than usual, because a specification must fit inside
  the model's context window. Read [docs/models.md](docs/models.md) before
  choosing one for real work.
