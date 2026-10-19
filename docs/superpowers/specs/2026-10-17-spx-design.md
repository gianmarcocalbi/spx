# spx — Design

**Date:** 2026-10-17
**Revised:** 2026-10-18, after a design interview
**Name:** spx ("superpowers-extended")

Items marked *(my call)* were decided by the writer of this document, not by
the product owner, and were all confirmed in the design interview. Everything
else was decided in the brainstorming conversation or in the interview.

## 1. Intent

spx is one plugin that sits on top of
[superpowers](https://github.com/obra/superpowers). It fits superpowers'
spec → plan → implementation flow to the way a scrum team works, without
forking superpowers or editing any of its files.

Superpowers assumes one person in one session. On a scrum team (SAFe, with
work assigned through JIRA), a single spec passes through many people over
months:

1. The PO and the experts brainstorm a feature idea and produce a **draft
   spec**. The draft may wait for months.
2. A dev picks it up and revises it with `/brainstorming`, giving the draft as
   input. The revision checks the draft against today's codebase and looks
   for gaps.
3. The dev takes the revised spec to a review meeting. There the experts and
   the PO challenge it, and the dev revises it again.
4. The dev says the spec is ready for review. spx opens a GitHub PR
   containing the spec, after the dev confirms it, and code owners review
   their parts. The spec is now **assessed**. The PR stays open.
5. The dev writes the plan with superpowers' `writing-plans`. The plan goes
   into the same PR. If the dev wants one, a short overview of the plan goes
   in too, and people review the overview instead of the plan.
6. The code lands in the same PR. When the PR merges, the spec is **done**.

Usually one consolidated spec has one plan. Sometimes a draft spec (a
"super-spec") leads to several specs, written in different sprints across
the PI, and each of those has its own plan.

spx adds what superpowers is missing for this:

- a status for every spec
- links between related specs, and to the JIRA tickets (or other items) a
  spec materializes
- spec sections organized by code owner
- a list of the decisions made while writing and revising the spec
- on request, a short overview of a plan that people can review
- a checker that reports when any of this drifts

The team's process is enforced outside spx, through JIRA, the review meeting
and the team's own rules. spx keeps the documents in step with that process
and stays out of its way.

**Audience.** spx is built for this team's flow first, and is publishable.
Team constants such as the 14-day window, the status names and the archive
folder are fixed in v1, not configurable. The README says so. Generalizing
waits for a second team.

## 2. Goals

- **Status:** every spec carries a status (`draft`, `assessed`,
  `in-progress`, `done`, `abandoned`). spx keeps it current at superpowers'
  natural moments, without anyone having to ask.
- **Spec content:** specs written or revised through superpowers'
  brainstorming get two additions:
  - a decisions list
  - an owner review section, with one subsection per affected code-owner
    area
- **Related specs:** specs can declare the specs they relate to. spx keeps
  those links valid.
- **Origins:** specs can list the JIRA tickets, or anything else, that they
  are a materialization of.
- **Spec PRs:** they request reviews from the GitHub accounts that own the
  affected areas.
- **Plans:** when the dev says yes, a plan gets an overview with diagrams,
  in a file next to it, which people review in the feature PR in place of
  the plan.
- **Checker:** a dependency-free checker reports drift in the specs, plans
  and overviews. The skills run it at fixed points, and it can also run in
  CI.
- **Setup:** a setup skill prepares a repo for spx, and later reviews or
  updates that setup.
- **Harnesses:** the same plugin works in Claude Code, GitHub Copilot CLI
  and Kiro CLI. Claude Code is the harness that gates the v1 release; the
  other two are verified best-effort (see 9).

## 3. Non-goals

- **Process:** spx doesn't run or track the review meeting, assign work, or
  detect that two devs picked up the same spec. JIRA handles assignment.
- **Revising specs:** that is superpowers' brainstorming. spx only makes sure
  brainstorming updates the existing file and keeps the spx parts.
- **Implementation decisions:** spx doesn't record decisions made during
  implementation, such as the `Ruling:` lines in superpowers' plan-execution
  ledger. Decisions come from writing and revising the spec only.
- **Approvals:** spx doesn't track PR approvals or record who approved what.
  The dev saying "the reviewers approved the spec" only refreshes
  `assessed-at` (7.5).
- **Spec readability:** specs are already written for people, so only plans
  get an overview.
- **Draft PRs:** spx doesn't open the PR for a draft spec. Drafts reach main
  through the team's normal flow. *(my call)*
- **Status from relations:** spx never works out a spec's status from the
  specs it relates to. A super-spec's status changes only when someone asks.
- **Cross-repo features:** a spec and the code that implements it live in the
  same repo. Features whose code spans several repos are out of scope for v1.
  *(my call)*

## 4. Anti-goals

spx has failed if any of these happen, even when everything else works:

- **spx blocks or gates a superpowers step once it is set up.** When the dev
  says "go on with the plan" or "start implementing", superpowers proceeds.
  spx may warn, and its checker may report, but it never refuses and never
  waits. The one gate is setup itself: without an ownership memory, spx
  stops and asks the dev to run setup (7.3).
- **spx produces an optional extra nobody asked for.** Extras such as the
  plan overview are offered, and made only when the dev says yes.
- **spx forks superpowers, edits its files, or restructures superpowers'
  spec content.** spx only adds to what superpowers writes.
- **spx depends on a harness-specific mechanism,** such as hooks. It must run
  on skills, instructions files and plain scripts alone.
- **The ownership memory is loaded every session,** or refreshed without
  anyone asking.
- **An agent plans or implements from an overview,** or fixes an overview
  without fixing the plan first.
- **spx grows into a process engine.** It must stay lean.

## 5. Constraints

- **Harnesses:** Claude Code, GitHub Copilot CLI and Kiro CLI.
  - Only portable building blocks: skills, instructions files, scripts, and
    agents if ever needed. No hooks.
  - The team installs superpowers in each tool itself. Superpowers' own
    support for Kiro is not spx's concern.
- **Neutral wording:** skill text describes actions in harness-neutral words,
  such as "read the file", "run the command", "send a helper agent" or "keep
  a task list". It never uses one tool's names, such as `Read`, `Task` or
  `TodoWrite`.
- **Packaging:** one plugin, made of several skills. Skill names carry the
  `spx-` prefix (`spx-setup`, `spx-spec-lifecycle`, `spx-illustrate-plan`):
  Kiro exposes every skill as `/<name>` and Copilot resolves skills by name,
  first found wins, so a bare `setup` would collide. In Claude Code they
  read `spx:spx-setup` and so on.
- **Superpowers is a prerequisite.** spx never modifies its files.
- **GitHub** is the forge. The GitHub CLI (`gh`) and git are the external
  tools spx uses.
- **Checker runtime:**
  - **Node only, for the checker.** The checker uses Node's built-in modules
    and nothing else.
  - **If a dependency ever becomes unavoidable** (a YAML parser, say), the
    skill tells people to install it globally. It never vendors anything and
    never adds an install step to the repo.
  - **Without Node,** the skills skip the check and say so. Nothing else in
    spx needs Node. Any current Node works; in CI it is the runner's.
  - **Arguments, not config.** The checker takes everything it needs as
    arguments: the specs folder, the plans folder and the adoption date. It
    never reads instructions files. The skills supply the arguments from
    their context, the CI workflow has them written in by setup, and without
    arguments the checker uses superpowers' default folders.
- **No MCP server** in v1. *(my call)*
- **English names:** spx's header fields and section headings are fixed and
  in English (`## Decisions`, `## Owner review`), so that the skills and the
  checker can find them. The prose in a spec can be in any language.
  *(my call)*

## 6. How superpowers can be extended (research findings)

The rest of this design rests on these findings.

- **Maintainers' guidance:** build a separate plugin, and steer superpowers
  through the repo's instructions files.
  - Superpowers' `using-superpowers` skill says: "User instructions
    (CLAUDE.md, AGENTS.md, GEMINI.md, etc, direct requests) take precedence
    over skills."
  - The maintainer confirmed in issue #125 that this is the way to extend
    superpowers.
  - No official extension points exist yet. Issue #2398 is open, with no
    design.
- **There is no spec template.** Brainstorming (v7) only lists what a spec
  must cover:
  - intent and why
  - goals, non-goals and anti-goals
  - constraints
  - the decided parts of the how
  - what is left to the builder

  Issue #2378 notes the missing template. Superpowers puts no front matter on
  specs, so spx owns that part of the file entirely.
- **Brainstorming saves a new file, and commits it.** (Verified in v7.0.0.)
  - Its only saving rule is
    `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`, committed, which
    the human partner's preferences override.
  - It never mentions revising an existing spec, so a revision session may
    write a second spec next to the draft.
  - The save and commit happen before its builder check and before the
    human approves the document.
  - Its HARD-GATE ends with "invoke superpowers:writing-plans and no other
    skill". That line fires after approval, not at save. So the save moment
    is free for an add-on; the post-approval moment is not.
- **writing-plans:** (verified in v7.0.0)
  - Saves to `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` (user
    preferences override this).
  - Every plan starts with a fixed header that has a `**Spec:**` line and a
    `## Review Focus` section.
  - Right after saving, in one message, it asks the dev to review the plan
    and choose how to run it. An add-on that wants a word first has to get
    it in before that message.
  - Its scope check may split a large spec into several plans.
- **finishing-a-development-branch:** (verified in v7.0.0)
  - Presents "exactly these 3 options": merge back into the base branch
    locally, push and create a PR following the repo's template, or keep the
    branch. None of them is "push to the existing PR", and the skill says to
    present the menu exactly as written.
  - It has no notion of a spec PR.
- **executing-plans never edits the plan.** Deviations become `Ruling:`
  lines in its ledger. Plans change before execution, when the dev revises
  them after feedback.
- **using-git-worktrees** may create a separate worktree and branch when
  execution starts.
- **How add-on skills fire on their own:** through a description written as
  "Use when/after X, before Y", combined with superpowers' rule that a skill
  must be used if it might apply.
  - A third-party spec-lifecycle add-on (spec-drift-mngr) reports its checks
    firing in 6 of 6 runs this way.
  - Another add-on (factory-gates) measured skills being crowded out by
    brainstorming's handoff, and the fixes were an instructions line or an
    explicit command. spx uses the instructions line (7.1).
- **Coupling risk:** an existing add-on pack
  (MarcelRoozekrans/superpowers-extensions) broke silently when v7 changed
  superpowers' wording. spx therefore lists every upstream touchpoint it
  relies on (8.2) and tests them against each new superpowers release.
- **Superpowers releases** are tagged `vX.Y.Z`, and the official Claude
  marketplace pins it by commit sha.
- **Harness facts** (from the official docs, 2026-10; not yet verified by
  running):
  - **Copilot CLI** loads `AGENTS.md`, `CLAUDE.md`,
    `.github/copilot-instructions.md` and a few more, all merged, walking
    from the working directory up to the git root. It expands `@path`
    imports in `AGENTS.md` and `CLAUDE.md` and dedupes identical files. It
    loads Agent Skills from `.github/skills/`, `.claude/skills/` and
    installed plugins, triggers them by description, installs Claude-style
    marketplaces directly (`copilot plugin marketplace add OWNER/REPO`), and
    dispatches generic subagents through a `task` tool.
  - **Kiro CLI** loads `.kiro/steering/*.md` and `AGENTS.md` (2.18+); it
    does not read `CLAUDE.md`. It loads skills only from `.kiro/skills/` and
    `~/.kiro/skills/`, triggers them by description, and exposes each as
    `/<name>`. Its plugin unit is a "Power": a root `plugin.json` following
    the Agent Plugins spec plus `skills/<name>/SKILL.md`. The CLI installs a
    Power from the catalog or a local path (2.25+, V3 engine). It has a
    `subagent` tool. Symlinked skills were reported ignored (issue #9787).
  - **Claude Code** reads `AGENTS.md` natively from v2.1.277 when no
    `CLAUDE.md` exists; otherwise `CLAUDE.md` imports it with an
    `@AGENTS.md` line. Its `plugin.json` has a `dependencies` field whose
    version ranges resolve against git tags named `<plugin>--v<version>`,
    which superpowers does not use.

## 7. Design

### 7.1 How spx hooks into superpowers

spx uses two portable mechanisms.

1. **An spx block in the repo's instructions file.** This is the backbone,
   because superpowers lets project instructions override its defaults.
   `spx-setup` writes the block between markers that carry spx's version and
   the adoption date (7.8):

   ```markdown
   <!-- spx:begin v1.0.0 adopted=2026-10-17 -->
   ...rules...
   <!-- spx:end -->
   ```

   Update mode replaces everything between the markers and keeps the
   adoption date.
2. **Skill descriptions that name superpowers' moments,** for example "Use
   when brainstorming saves or revises a spec…" or "Use after writing-plans
   saves a plan…".

The skills carry the full rules. The block is a pointer to them, not a copy.
Superpowers' own rule is that instructions files beat skills and skills beat
defaults, so a skill can never override another skill's explicit text. That
is the test for every line of the block: it carries the triggers, the rules
that contradict a superpowers skill's wording, and the one guardrail that
sessions with no spx skill loaded still need. Everything additive lives in
the spx skills, in one place each. The block must say, in substance (the
builder chooses the wording and tests it):

- **Trigger:** when brainstorming, writing-plans or
  finishing-a-development-branch starts in this repo, also use
  `spx-spec-lifecycle` and keep it active for the session.
- **Revising:** when brainstorming starts from an existing spec, revise that
  file in place. (Contradicts brainstorming's save to a new dated file.)
- **Plan overview:** after writing-plans saves a plan and before its review
  question, ask the dev whether they want an overview for people, and use
  `spx-illustrate-plan` on a yes. (Has to land before writing-plans' own
  post-save message.)
- **Overviews:** `.overview.md` files next to plans are for people. Plan and
  implement from the plan and the spec. Feedback on an overview is feedback
  on the plan: fix the plan, and the spec if needed, then regenerate the
  overview with `spx-illustrate-plan`. After changing a plan that has an
  overview, offer to regenerate it. (The guardrail: execution sessions and
  subagents read this with no spx skill loaded.)
- **Finishing:** when an open PR exists for the current branch, skip
  finishing's menu: run spx's finishing step, commit, and push to that PR.
  In a worktree, merge into the PR's branch first. (Contradicts finishing's
  "present exactly these 3 options".)

Moved out of the block and into `spx-spec-lifecycle`, where they are
additive: keeping the front matter, decisions and owner review on a
revision; doing nothing when the session is not writing a spec; and the
ownership memory being read only at spx's moments, which is probably a
no-op as an instruction and is tested as one (11).

**Which instructions file.** One copy of the block, in `AGENTS.md` at the
repo root. All three harnesses read it: Copilot CLI and Kiro always, Claude
Code natively from v2.1.277 when no `CLAUDE.md` exists. When a `CLAUDE.md`
exists and does not already import `AGENTS.md`, setup adds an `@AGENTS.md`
line to it, which Copilot also expands and dedupes. Setup never writes to
`.github/copilot-instructions.md` or `.kiro/steering/`. The README states
the minimum Claude Code version.

**Sessions started above the repo.** A session started in a parent folder
that holds several repos may not load a repo's instructions file at all
(Copilot, for one, walks upward from the working directory, so a repo below
it is never on the path). For that reason:

- the skill descriptions and skill bodies carry everything spx needs, so spx
  still works without the block, only with less reliable triggering
- spx's README recommends starting sessions inside the repo *(my call)*

### 7.2 Skills

| Skill | Fires | Purpose |
|---|---|---|
| `spx-setup` | Only when the dev types it; the gate (7.3) asks them to | Prepare, review or update a repo for spx, including the ownership memory |
| `spx-spec-lifecycle` | Loaded when brainstorming, writing-plans or finishing starts in the repo (7.1); acts at spec moments | Spec format, related specs, status changes, spec PR, archiving; keeps the owner review and decisions consistent; ships the checker |
| `spx-illustrate-plan` | Only when the dev asks, or says yes when offered after a saved plan | Write the plan's overview |

- **No separate ownership skill.** "Review code ownership" means running
  `spx-setup` in update mode with the ownership step selected. *(my call)*
- **No plugin agents in v1.** When a skill needs a helper, it hands a generic
  subagent a prompt file shipped with the skill, the way superpowers does.
  Agent definition formats differ from harness to harness, and all three
  harnesses can dispatch a generic subagent (Claude Code's agent tool,
  Copilot's `task` tool, Kiro's `subagent` tool). *(my call)*
- **Invocation.** `spx-setup` is user-invoked (`disable-model-invocation:
  true` where the harness has it). It only ever fires by hand, so it pays
  no context load, and the gate (7.3) asks the dev to type `/spx-setup`
  instead of invoking it. Kiro has no such flag, so its description must
  read safely there: "Use when the dev asks to set up spx, review its setup,
  or update it after an upgrade or an ownership change." The other two are
  model-invoked: the agent reaches `spx-illustrate-plan` after the dev's
  yes, and `spx-spec-lifecycle` from the block and from its own description.
- **Descriptions are pointers.** Each lists its trigger branches, one
  trigger per branch, leading word first, with no synonyms and nothing the
  body already says. `spx-spec-lifecycle`'s branches are the moments in 7.5.
  `spx-illustrate-plan`'s are: yes to the offer, a direct request, a refresh
  after the plan changed. Superpowers' "invoke if there is a 1% chance it
  applies" rule makes pointers fire eagerly, so tight beats broad.
- **Layout of `spx-spec-lifecycle`.** It loads at the start of every
  brainstorming session, so its `SKILL.md` holds only what every session
  needs: the moment table and the spec-writing rules (7.4). Each other
  moment (ready for review, approval, planning, finishing, abandon, split,
  the checker how-to) is a sibling file reached when that moment arrives.
- **Helpers.** The consistency pass (7.4) and the overview generation (7.7)
  are subagent dispatches with prompt files. A fresh context reads the spec
  against the memory without the session's drift, and a renderer that only
  writes the overview cannot quietly fix the plan. Without a subagent tool,
  the skill does the step itself.

### 7.3 Ownership memory

- **Location:** `docs/spx/ownership.md`, committed. *(my call)*
  - `docs/spx/` is spx's home folder in each repo.
  - None of the three harnesses loads `docs/` on its own.
  - It must never go somewhere loaded automatically, such as `CLAUDE.md` or
    `.kiro/steering/`.
- **Format:** Markdown, one block per area *(my call)*:

  ```markdown
  ## Security
  - **Owners:** @org/security, @alice
  - **Scope:** authentication, secrets, anything handling PII; consult before touching card flows
  - **Paths:** `src/auth/**`, `src/payments/card/**`, `config/secrets/**`
  ```

- **Sources:**
  - **Primary:** the CODEOWNERS file of the repo the spec is written into.
    That is always the repo where the spec lands, even when the session
    started in a folder above several repos. GitHub looks for CODEOWNERS in
    `.github/`, the root, and `docs/`.
  - **Comments in CODEOWNERS count.** They name areas and describe their
    scope, for example
    `# Payments team — PCI scope, talk to them before touching card flows`.
  - **Fallback:** if there is no CODEOWNERS, or it says too little, spx looks
    for ownership information elsewhere: instructions files, READMEs and
    contributing docs.
  - **Confirmation:** in every case, spx shows the user the result, and the
    user confirms or corrects it before it is saved.
- **Owners** are GitHub accounts: `@user` or `@org/team`.
- **Missing memory, the one gate:** when a spec moment arrives and
  `docs/spx/ownership.md` does not exist, `spx-spec-lifecycle` stops and
  asks the dev to run `/spx-setup` (init mode), then continues with the
  spec once it has. There is no "proceed without". Sessions that are not
  writing a spec never reach this gate.
- **Refreshing:** only through `spx-setup` (update mode, ownership step).
  spx never checks or refreshes the memory on its own.
- **Note, not decided:** the Owners and Paths lines are a cache of
  CODEOWNERS and go stale between setup runs (see Risks). The lines
  CODEOWNERS cannot say, the area names and the scope prose, are what earns
  the memory its place. An alternative is to keep only those in the memory
  and read handles and paths from CODEOWNERS at the moment of use. The
  decided design stands; this is recorded for a later revision.
- **Readers:**
  - `spx-spec-lifecycle`, for owner sections and reviewers
  - `spx-illustrate-plan`, to tag each task with its owner area
  - `spx-setup`, which builds and reviews it

  Nothing else reads it.

### 7.4 Spec format

spx adds to superpowers' spec. It never restructures it. Superpowers'
content stays exactly as brainstorming writes it, and spx adds three parts:

```markdown
---
status: assessed
assessed-at: 2026-10-17T14:05:00Z
materializes:
  - SHOP-1423
  - https://example.atlassian.net/wiki/spaces/SHOP/pages/98765
relates-to:
  - docs/superpowers/specs/2026-06-02-checkout-design.md
---
# <Feature> Design

<superpowers' usual content, untouched>

## Decisions
- D1 (2026-10-17) Use X over Y — because … — areas: backend, security

## Owner review
Not affected: frontend, devops

### Security — @org/security
Scope: authentication, secrets, PII handling
- What changes for you: …
- Decisions that touch you: D1, D4
- Open questions and risks: …
```

**Front matter fields:**

| Field | Required | Meaning |
|---|---|---|
| `status` | Always | `draft`, `assessed`, `in-progress`, `done` or `abandoned` |
| `assessed-at` | Once the spec has been assessed | UTC timestamp (ISO 8601) of when it became `assessed`. Reset on every revision while it is still `assessed`, and when the dev says the reviewers approved the spec. Drives the 14-day flag (7.5) |
| `materializes` | Optional | What the spec is a materialization of: JIRA keys (`SHOP-1423`) or links to anything else, such as a wiki page. The field name is *(my call)* |
| `relates-to` | Optional | Repo-relative paths to related specs |

- **`materializes`:**
  - spx records the tickets and links the dev mentions. When a new spec is
    saved without any, spx asks once; "none" is a fine answer. *(my call)*
  - spx never looks anything up in JIRA or elsewhere. The entries are
    references, nothing more.
  - When a spec is created from another one, spx asks which items the new
    spec materializes instead of copying them over. *(my call)*
- **Format rule:** only `key: value` lines and multi-line lists (`- item`,
  one per line).
  - No inline lists like `[a, b]`.
  - No nested structures.
  - No multi-line strings.

  This is what lets the checker read the header without a YAML library.
- **No status history in the file.** Git is the history. A revision uses git
  to find what changed in the codebase since the spec was last committed.
  *(my call)*

**Decisions:**

- The list holds every decision made while writing or revising the spec.
- Each decision has its date, its reason, and the owner areas it touches.
- Decisions are numbered, so owner sections can point to them.
- A revision adds to the list. When a decision is reversed, the old entry is
  marked superseded, never deleted.

**Owner review:**

- **Which areas:** the areas come from the ownership memory.
  `spx-spec-lifecycle` judges which ones the spec affects, from the paths and
  components it touches and from the scope of each area.
- **Unaffected areas:** they are listed in one "Not affected" line, so their
  owners can push back.
- **Affected areas:** each gets a subsection containing:
  - its GitHub owners
  - its scope line
  - what changes for that area
  - the decisions that touch it
  - its open questions and risks
- **What a section says:** only what that owner must sign off on. It points
  to the main text for details instead of repeating them.
- **What a section may add:** matters that belong to one area and have no
  natural place in the main text (such as a threat consideration) go in that
  area's section.

**Free-form sections:**

- Inside `## Decisions` and `## Owner review`, only the headings and the
  `D<n>` numbering are fixed. The lines themselves are prose; the example
  above shows one shape, not a format.
- The checker does not parse inside the two sections (7.8).
  `spx-spec-lifecycle` keeps them consistent itself, reading them in free
  form: areas and owners against the ownership memory, decision references
  against the list. It does this whenever it saves or revises the spec and
  before opening or updating the spec PR, fixes what it finds, and tells the
  dev.

**Location:**

- Specs live wherever superpowers saves specs in that repo (default
  `docs/superpowers/specs/`).
- Specs that are `done` or `abandoned` move to an `archive/` subfolder there.
- Specs that are `draft`, `assessed` or `in-progress` never move.

### 7.5 Lifecycle

There are five statuses. There is no "outdated": a stale spec goes back to
`draft`. `assessed` means *submitted for assessment*: it is set when the dev
says the spec is ready for review, before anyone has reviewed it. spx records
no approvals (see Non-goals); the dev saying "the reviewers approved the
spec" only refreshes `assessed-at`.

| Moment | What spx does | Where the change lives |
|---|---|---|
| Brainstorming saves a new spec | `status: draft` | Merged to main on its own, through the team's flow |
| The dev says the revised spec is ready for review | `status: assessed`, `assessed-at` = now; spx opens the spec PR after the dev confirms it | Feature branch / PR |
| The spec is revised while `assessed` (e.g. after review comments) | `assessed-at` reset to now; reviewers added for any newly affected area | Same PR |
| The dev says the reviewers approved the spec | `assessed-at` = now; nothing else is recorded | Same PR |
| "Ready for review" again while the PR is open | Update the existing PR; never open a new one | Same PR |
| writing-plans saves a plan from the spec | `status: in-progress`; confirm the plan's `**Spec:**` line; offer the overview, written by `spx-illustrate-plan` only on a yes; commit and push the plan (and the overview, if any) to the PR before any code | Same PR |
| The finishing step | `status: done` (see "Several plans" below); spec moved to `archive/`; links fixed | Same PR, before merge |
| Someone says the feature is dropped | `status: abandoned`; spec moved to `archive/`; links fixed | Wherever the spec is |

**Every moment ends on a checkable bound.** A moment's step is done when a
criterion that names every item holds, never when enough has been done. The
"Not affected" line already does this for the owner review: every area of
the memory accounted for exactly once. The others take the same shape: every
decision made in the session is in `## Decisions`; every affected area's
owners are requested on the PR; every `relates-to` entry and `**Spec:**`
line pointing at an archived spec is updated; the checker has run and its
report is shown.

**Where statuses show up.** Main only ever shows `draft`, `done` or
`abandoned`. `assessed` and `in-progress` exist on the feature branch. So
for as long as the PR is open, main shows `draft` for a spec that is under
review or implementation. That is accepted: JIRA is where "is this taken"
is answered (see Non-goals).

**Every status change is committed.**

**The 14-day rule.**

- An `assessed` spec is trusted for 14 days from `assessed-at`. The window
  is fixed in v1.
- Whenever `spx-spec-lifecycle` meets an `assessed` spec past that point
  (notably when planning starts from it), it warns that the assessment may
  no longer hold and sets the spec back to `draft`.
- It doesn't block. If the dev carries on and a plan is saved, the status
  becomes `in-progress` as usual.

**Opening the spec PR.**

- **Status:** set `assessed` and `assessed-at`, then commit.
- **Branch:** push the current branch. If the dev is on the default branch,
  ask for a branch name first, since teams name branches after JIRA keys.
  *(my call)*
- **PR:** open it with `gh` and fill in the repo's PR template. If the
  template asks for tickets, take them from `materializes`.
- **Confirmation:** creating a PR is outward-facing, so spx shows the title,
  the body and the reviewer list, and runs `gh pr create` only after the dev
  says yes.
- **Reviewers:** request the GitHub owners of every affected area, taken
  from the ownership memory. spx has to do this explicitly. GitHub only
  requests owners automatically for the paths a PR changes, and a spec PR
  only touches the specs folder.
- **Nothing else.** spx adds nothing more to the PR. Its description is the
  team's template. The overview is reviewed by whichever spec reviewers the
  dev asks. spx doesn't re-request reviews when the plan is pushed.
  *(my call)*

**Finishing.**

- **No second PR:** when an open PR exists for the spec, finishing never
  opens another one.
- **Finishing's menu is overridden.** finishing-a-development-branch offers
  exactly three options, merge locally, push and create a PR, or keep the
  branch, and none of them is "push to the existing PR". The block (7.1)
  says: when an open PR exists for the current branch, skip the menu, run
  spx's finishing step, commit, and push to the PR.
- **Worktrees:** if superpowers created a separate worktree branch for
  execution, finishing merges it into the PR's branch first, then pushes.
- **Several plans:** if more than one plan points to the spec, spx asks the
  dev whether the spec is now complete. If so, it sets `done`. If not, it
  stays `in-progress`. *(my call)*

**Off the usual path.** spx never blocks, so it warns and goes on. *(my
call)* Examples:

- planning from the default branch
- planning without a spec PR

The status is still recorded, and the checker reports the result if it ends
up on main.

**No gating, no approval tracking.** If the dev asks for the plan or for
implementation, superpowers proceeds. spx only keeps the documents right.
The only gate is setup (7.3).

**Abandoning.** spx sets the status, moves the spec and fixes the links.
Closing an open feature PR, and getting the change to main, follow the
team's usual flow.

### 7.6 Related specs and super-specs

- **`relates-to`** links a spec to other specs.
  - The links are untyped: spx doesn't model parent and child.
  - When spx creates a spec from another one, for example by splitting a
    super-spec in a later sprint, it adds the link in both directions so that
    each side finds the other. *(my call)*
- **Usual case:** one consolidated spec with one plan.
- **Super-spec:** a draft can lead to several specs, created in different
  sprints across the PI. Each has its own lifecycle, its own plan, and its
  own PR.
- **A super-spec's status** changes only when someone asks. The checker gives
  a hint when a `draft` spec's related specs are all `done` or `abandoned`,
  so nobody forgets to close it. The hint changes nothing, and the dev sees
  it at finishing, when the skills run the checker. *(my call)*
- **Moves:** when a spec moves to `archive/`, spx updates every `relates-to`
  entry and `**Spec:**` line that points to it.

### 7.7 Plan overview (`spx-illustrate-plan`)

- **Why:** plans are long, detailed and technical. People review the overview
  instead: first the dev, then one or more of the spec's reviewers, in the
  feature PR, before any code is pushed.
- **When:** it never runs on its own.
  - After writing-plans saves a plan, spx asks the dev whether they want an
    overview, before superpowers' plan review question. It writes one only
    on a yes. On a no, spx doesn't ask again for that plan.
  - Whenever someone asks for it, optionally naming what to show.
- **Where:** next to the plan, with the same name plus `.overview`:

  ```
  docs/superpowers/plans/2026-10-17-checkout-redesign.md            ← plan (for agents)
  docs/superpowers/plans/2026-10-17-checkout-redesign.overview.md   ← overview (for people)
  ```

  Plans and overviews never move. Only the spec moves to `archive/`.
- **One word:** the file is an *overview* everywhere: in the block, the
  skill description, the suffix, the first line and the skill bodies. The
  word "sidecar" is retired, so the agent links one token to one rule.
- **First line:** it names the file for what it is:
  *"Overview for people of `<plan path>`. Agents plan and implement from the
  plan itself."*
- **Who writes it:** a subagent given the skill's prompt file, the plan, the
  spec and the ownership memory (7.2). It renders and nothing else.
- **Default contents:**
  1. **What gets built, in what order:** a few lines in plain language.
  2. **Context map:** which parts of the codebase change, which are new,
     and which are untouched.
  3. **Task map:** the tasks in order, their dependencies, and the owner area
     each one touches (from the ownership memory).
  4. **Spec coverage:** each spec requirement and decision, mapped to the
     task(s) that carry it out. This lets a reviewer check that everything
     is covered and nothing extra slipped in.
  5. **Decisions worth a human look:** choices the plan made that the spec
     didn't settle, and that are hard to undo or affect other people. For
     example: new public interfaces, new dependencies, data migrations,
     irreversible steps.
  6. **Review Focus:** the plan's own `## Review Focus` list, brought up
     front.
- **Extras on request:**
  - data changes (tables, fields, relations, migrations)
  - main flow
  - trust boundaries
  - state machine
  - rollout (flags, migration order, rollback)
- **Format:** Mermaid diagrams. GitHub renders them, all three tools can
  write them, and nobody needs to install anything. *(my call)*
- **Keeping it in step:** the block (7.1) says that after changing a plan
  that has an overview, the agent offers to regenerate it. Since
  executing-plans never edits plans, this happens before execution, when the
  dev revises the plan after feedback. The checker's staleness report at
  finishing is the safety net. A plan problem is fixed in the plan, and the
  overview is regenerated from it.

### 7.8 Checker

- **What it is:** a script shipped inside `spx-spec-lifecycle`.
  - Plain Node, built-in modules only (see Constraints).
  - It only reports. It never edits files and never stops anything.
  - **Arguments:** the specs folder, the plans folder and the adoption date
    (7.1). Without arguments it uses superpowers' default folders. It never
    reads instructions files. Inside the folders it treats every `.md` file
    as a spec or plan, `.overview.md` files as overviews, and `archive/` as
    the archive.
- **What it checks:**

  | Area | Check | Severity |
  |---|---|---|
  | Front matter | Present and within the allowed format; inline lists flagged | Error |
  | Front matter | `status` is one of the five values | Error |
  | Front matter | `assessed-at` is a valid timestamp, and present when `status: assessed` | Error |
  | Front matter | `materializes` entries are non-empty (never looked up) | Warning |
  | Staleness | An `assessed` spec past 14 days | Warning |
  | Links | Every `relates-to` entry points to an existing spec | Error |
  | Sections | `## Decisions` and `## Owner review` are present | Error |
  | Location | `done` and `abandoned` specs are in `archive/`; other specs are not | Error |
  | Branch | On the default branch, only `draft`, `done` or `abandoned` (only when run inside git) | Warning |
  | Hint | A `draft` spec whose related specs are all `done` or `abandoned` | Hint |
  | Plans | Every plan's `**Spec:**` line points to an existing spec | Error |
  | Overviews | Every `.overview.md` overview has its plan next to it (a plan without an overview is fine) | Error |
  | Overviews | The first line names its plan | Error |
  | Overviews | The overview is not older than its plan (by last commit, else by file time) | Warning |
  | Legacy | Count of legacy files skipped | Hint |

- **What it does not check:** anything inside the two sections. Consistency
  with the ownership memory and between decisions and their references is
  `spx-spec-lifecycle`'s job (7.4).
- **Legacy files:** a spec or plan is legacy when its date is before the
  adoption date and it has no spx front matter. The date is the filename's
  `YYYY-MM-DD` prefix; without one, the file's first commit date from git;
  without git, the file counts as legacy. Legacy files are skipped by every
  check and only counted. A pre-adoption spec that someone later revises
  gets front matter from `spx-spec-lifecycle` and is then checked like any
  other. There is no migration step.
- **Output:** a short report to read, with errors, warnings and hints.
  - The exit code is non-zero only on errors, so CI can show a failing check.
  - Whether that failure blocks a merge is the team's choice, not spx's.
- **When the skills run it:**
  - after a spec is saved or revised
  - before opening or updating the spec PR
  - after a plan is saved (and its overview, if one was asked for)
  - at finishing, after archive moves

  The skills show the dev the report and carry on.
- **In CI:** setup can add a workflow that runs it on PRs through spx's
  composite action (7.9, step 6, and 8.1).

### 7.9 Setup (`spx-setup`)

- **It runs only when the dev types it,** on their own or because the gate
  (7.3) asked them to, and it goes through numbered steps.
- **When called, it asks two things:**
  - **Mode:** `init` by default if the repo has no spx setup yet (no spx
    marker in `AGENTS.md` and no `docs/spx/ownership.md`). Otherwise
    `update/review` by default.
  - **Steps:** all of them by default.

**The steps:**

1. **Prerequisites:** superpowers is available; `gh` is installed and signed
   in; Node is available (only the checker needs it).
2. **CODEOWNERS:** exists, uses GitHub accounts, and has comments describing
   each area. Setup reports the gaps and offers a draft for the user to
   approve.
3. **Ownership memory:** build it (init), or review it against the current
   CODEOWNERS and the other sources (update). Always confirm with the user
   before saving.
4. **Instructions file:** write or replace the versioned spx block in
   `AGENTS.md`, with the adoption date in the marker (set at init, kept on
   update). If a `CLAUDE.md` exists without an `@AGENTS.md` import, add the
   line (7.1).
5. **PR template:** check that one exists, and offer a starter for spec PRs
   if it doesn't.
6. **CI check (optional):** offer a GitHub Actions workflow that runs the
   checker on PRs through spx's composite action, pinned to spx's version
   tag, with the folders and the adoption date as inputs (8.1).
   - No copy of the checker lives in the team's repo.
   - Update mode bumps the tag.
7. **Summary and commit:** show every change, then commit after the user
   confirms.

Setup never touches repo settings on GitHub, such as branch rules. Those
belong to the team's process. *(my call)*

**Update/review mode.** Each step compares what's in the repo with what spx
expects, reports the differences, and proposes fixes. It doesn't rebuild
from scratch. Re-running it after an spx upgrade refreshes the block and the
action tag in the workflow.

### 7.10 Team setup list (for spx's README)

To get the most out of spx, a team needs:

1. superpowers and spx installed in every tool the team uses
2. CODEOWNERS using GitHub accounts, with comments describing what each area
   covers
3. a PR template for spec PRs
4. the GitHub CLI installed and signed in wherever spx runs, and Node for the
   checker
5. optionally, the checker running in CI through spx's action, which needs
   the org-wide Actions access setting on the spx repo (done once by
   whoever hosts spx in the org)
6. `spx-setup` run once in init mode, then re-run in update mode when
   ownership changes or spx is upgraded
7. sessions started inside the repo, not in a parent folder

## 8. Packaging and upstream

### 8.1 Packaging

- **Repo:** one repo, `spx`, hosted in the company org and reachable from
  the team's CI. The public `gianmarcocalbi/spx` becomes a mirror or is
  retired; release tags live in the repo the action is used from.
- **Two manifests, one `skills/` folder:**
  - `.claude-plugin/plugin.json` plus a marketplace, for Claude Code and
    Copilot CLI, which installs Claude-style marketplaces directly.
  - a root `plugin.json` in the Agent Plugins format, so Kiro users clone
    the repo and install it as a Power from the local path.
  - A release script keeps the version in both. The README documents the
    fallback for Kiro: copy `skills/*` into `~/.kiro/skills/`.
- **Kiro:** the team installs superpowers in Kiro itself. spx never edits
  superpowers' agent file.
- **Composite action:** the repo ships `.github/actions/check/action.yml`,
  which runs the checker with the folders and the adoption date as inputs.
  Team workflows use it at a pinned tag. The spx repo's Actions access
  setting allows the org's repos to use it, so no token is needed.
- **Superpowers version:**
  - Claude Code's `plugin.json` declares superpowers as a dependency *by
    name only* (`superpowers@claude-plugins-official`). A version range would
    not resolve: Claude Code matches ranges against `<plugin>--v<version>`
    tags and superpowers tags `vX.Y.Z`. Copilot ignores the field; Kiro has
    none.
  - The README states the tested version range as a prerequisite. The
    contract test (8.2) is the real guard.

### 8.2 Upstream contract

These are all the parts of superpowers that spx depends on:

- **Skill names:** `brainstorming`, `writing-plans`,
  `finishing-a-development-branch`, `using-git-worktrees`,
  `subagent-driven-development`, `executing-plans`.
- **Default locations and names:**
  `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` and
  `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`.
- **Overridable locations:** the "preferences override" wording for both
  locations.
- **Plan header:** the `**Spec:**` line and the `## Review Focus` heading.
- **Instructions win:** `using-superpowers`' rule that user instructions take
  precedence over skills.
- **Finishing:** finishing-a-development-branch merges back into the branch
  the work came from, uses the repo's PR template, and presents exactly
  three options with the "present the menu exactly as written" rule.
- **Brainstorming's flow:** it commits the spec at save, and its HARD-GATE
  hands off to writing-plans after approval.
- **writing-plans' handoff:** the single post-save message that asks for
  plan review and the execution approach.

spx's own CI runs a contract test against superpowers' latest release. The
test fails when any of these disappears or changes, so the break is caught
before teams hit it.

## 9. Acceptance

spx's skills shape agent behavior, so they are tested as behavior. Each test
uses ordinary prompts and never names an spx skill, except `spx-setup` and
on-request `spx-illustrate-plan`.

**Claude Code gates the v1 release:** every scenario below must pass there.
Copilot CLI and Kiro are verified best-effort, and the README states what
passed where.

In Kiro, the tests count only once superpowers' own check passes there:
"Let's make a react todo list" must trigger brainstorming.

1. **New spec:** brainstorming a new feature produces a spec with
   `status: draft`, a decisions list, and an owner review section that
   matches the ownership memory. A JIRA key mentioned in the conversation
   lands in `materializes`; with none mentioned, spx asks once.
2. **Revision:** brainstorming with an existing draft as input updates that
   file in place, adds decisions, and creates no second spec.
3. **Ready for review:** saying "the spec is ready for review" sets
   `assessed` and `assessed-at`, shows the PR for confirmation, and opens a
   PR whose reviewers are the affected areas' owners. Saying it again
   updates the same PR.
4. **Revision under review:** revising an `assessed` spec resets
   `assessed-at` and adds reviewers for newly affected areas.
5. **Stale spec:** planning from an `assessed` spec more than 14 days past
   `assessed-at` warns and resets it to `draft`, without blocking the plan.
6. **Plan saved:** the spec becomes `in-progress`, and the dev is asked
   whether they want an overview before the plan review question. On a yes,
   an `.overview.md` overview with the default contents appears; on a no, none
   is written and the question isn't repeated. The plan, and the overview if
   any, are pushed to the PR.
7. **Finishing:** the spec becomes `done` and moves to `archive/`. Its
   `relates-to` and `**Spec:**` links are fixed, and the work goes into the
   existing PR. Finishing's menu is not shown, and no new PR is opened,
   including when a worktree was used.
8. **Splitting a super-spec:** creating a spec from a draft super-spec links
   the two in both directions, and the super-spec's status stays unchanged.
9. **Checker:** the checker reports each check in 7.8 on purpose-built bad
   fixtures, passes clean ones, ignores older files, and runs with no
   installed packages.
10. **Setup:** `init` on a bare repo, `update/review` on a set-up repo, and
    step selection all behave as in 7.9.
11. **Overviews:** asked to implement the plan, the agent works from the plan
    and the spec, not the overview.
12. **Setup gate:** brainstorming a spec in a repo with no
    `docs/spx/ownership.md` stops and asks the dev to run `/spx-setup`;
    after it runs, the spec is written with its owner review section.
    Brainstorming something that is not a spec never hits the gate.
13. **Approval reset:** saying "the reviewers approved the spec" resets
    `assessed-at` and changes nothing else.
14. **Instructions file:** setup on a repo with a `CLAUDE.md` writes the
    block once, in `AGENTS.md`, and adds the `@AGENTS.md` import; on a repo
    without one it writes only `AGENTS.md`.
15. **Consistency pass:** after the ownership memory changes, revising an
    `assessed` spec updates its owner review section and reports the
    change; a reference to a decision that does not exist is caught.

The builder may use superpowers' eval harness (superpowers-evals) or plain
scripted sessions. The checker and the contract test are ordinary automated
tests.

## 10. Risks

- **Brainstorming's handoff.** Its "invoke writing-plans and no other skill"
  fires after approval, and `spx-spec-lifecycle` is loaded at the start of
  brainstorming and does its save-time work before brainstorming's own
  commit, so the handoff no longer crowds it out. The residual risk is
  instruction adherence: scenario 1 must prove the block loads the skill,
  and the checker reports a non-legacy spec with no front matter if it
  slips through.
- **Brainstorming can ignore instructions about where outputs go** (open
  issue #939). Scenario 2 must prove in-place revision. The checker can't
  reliably detect a duplicate spec, so this risk stays partly open.
- **Upstream changes:** caught by the contract test (8.2).
- **Copilot CLI and Kiro are verified from their docs, not by running.**
  Both load `AGENTS.md` and trigger skills by description on paper. Kiro
  reportedly ignores symlinked skills, so its install is tested with real
  copies. Kiro's custom agents may not inherit steering and skills (its docs
  disagree); spx uses no custom agents, so this only matters for teams that
  do.
- **Drift between sessions is not caught by CI.** The consistency of the
  owner review with the ownership memory is checked by the skill, not the
  checker (7.4), so a memory change only reaches a spec the next time spx
  touches it.

## 11. Left to the builder

- **Wording:** the exact wording of the skill descriptions and the
  instructions block. Develop it test-first with
  `superpowers:writing-skills`, under these rules:
  - state the target behaviour, not the prohibition ("plan from the plan",
    "revise the existing file", "push to the open PR"); keep a prohibition
    only as a guardrail, paired with its positive line
  - one word per concept, the same token in the block, the descriptions,
    the file names and the skill bodies: *overview*, *spec moment*,
    *ownership memory*, *legacy*, the five status names
  - every line passes the no-op test: it changes behaviour against the
    agent's default, or it goes
  - each meaning lives in one place; the block points, the skills hold
- **Formats:** the `ownership.md` layout and the Mermaid details.
- **Action and workflow:** the composite action's inputs and the starter
  workflow setup offers (7.9, 8.1).
- **Release script:** how the two manifests and the tag stay in step (8.1).
- **Layout:** the internal file layout of the plugin and of each skill,
  within 7.2's rule for `spx-spec-lifecycle`.
