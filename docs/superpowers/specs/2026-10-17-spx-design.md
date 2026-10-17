# spx — Design

**Date:** 2026-10-17
**Name:** spx ("superpowers-extended")

Items marked *(my call)* were decided by the writer of this document, not by
the product owner. Everything else was decided in the brainstorming
conversation.

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
4. The dev opens a GitHub PR containing the spec, and code owners review
   their parts. The spec is now **assessed**. The PR stays open.
5. The dev writes the plan with superpowers' `writing-plans`. The plan and a
   short overview of it go into the same PR. People review the overview, not
   the plan.
6. The code lands in the same PR. When the PR merges, the spec is **done**.

Usually one consolidated spec has one plan. Sometimes a draft spec (a
"super-spec") leads to several specs, written in different sprints across
the PI, and each of those has its own plan.

spx adds what superpowers is missing for this:

- a status for every spec
- links between related specs
- spec sections organized by code owner
- a list of the decisions made while writing and revising the spec
- a short overview of each plan that people can review
- a checker that reports when any of this drifts

The team's process is enforced outside spx: JIRA, the review meeting, and
GitHub branch protection. spx keeps the documents in step with that process
and stays out of its way.

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
- **Spec PRs:** they request reviews from the GitHub accounts that own the
  affected areas.
- **Plans:** every plan gets an overview sidecar with diagrams, which people
  review in the feature PR in place of the plan.
- **Checker:** a dependency-free checker reports drift in the specs, plans
  and sidecars. The skills run it at fixed points, and it can also run in CI.
- **Setup:** a setup skill prepares a repo for spx, and later reviews or
  updates that setup.
- **Harnesses:** the same plugin works in Claude Code, GitHub Copilot CLI
  and Kiro CLI.

## 3. Non-goals

- **Process:** spx doesn't run or track the review meeting, assign work, or
  detect that two devs picked up the same spec. JIRA handles assignment.
- **Revising specs:** that is superpowers' brainstorming. spx only makes sure
  brainstorming updates the existing file and keeps the spx parts.
- **Implementation decisions:** spx doesn't record decisions made during
  implementation, such as the `Ruling:` lines in superpowers' plan-execution
  ledger. Decisions come from writing and revising the spec only.
- **Approvals:** spx doesn't track PR approvals or record who approved what.
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

- **spx blocks or gates a step.** When the dev says "go on with the plan" or
  "start implementing", superpowers proceeds. spx may warn, and its checker
  may report, but it never refuses and never waits.
- **spx forks superpowers, edits its files, or restructures superpowers'
  spec content.** spx only adds to what superpowers writes.
- **spx depends on a harness-specific mechanism,** such as hooks. It must run
  on skills, instructions files and plain scripts alone.
- **The ownership memory is loaded every session,** or refreshed without
  anyone asking.
- **An agent plans or implements from a sidecar overview,** or fixes a
  sidecar without fixing the plan first.
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
- **Packaging:** one plugin, made of several skills.
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
    spx needs Node.
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
- **Brainstorming saves a new file.**
  - Its only saving rule is
    `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`, which the human
    partner's preferences override.
  - It never mentions revising an existing spec, so a revision session may
    write a second spec next to the draft.
- **writing-plans:**
  - Saves to `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` (user
    preferences override this).
  - Every plan starts with a fixed header that has a `**Spec:**` line and a
    `## Review Focus` section.
  - After saving, it asks the dev to review the plan and choose how to run
    it.
  - Its scope check may split a large spec into several plans.
- **finishing-a-development-branch:**
  - Merges back into the branch the work came from, or pushes and opens a PR
    following the repo's PR template.
  - It has no notion of a spec PR.
- **using-git-worktrees** may create a separate worktree and branch when
  execution starts.
- **How add-on skills fire on their own:** through a description written as
  "Use when/after X, before Y", combined with superpowers' rule that a skill
  must be used if it might apply.
  - A third-party spec-lifecycle add-on (spec-drift-mngr) reports its checks
    firing in 6 of 6 runs this way.
  - **Known risk:** brainstorming ends with "invoke superpowers:writing-plans
    and no other skill". That can crowd out a skill that should run right
    after a spec is saved. Another add-on (factory-gates) measured this, and
    the fixes were an instructions line or an explicit command.
- **Coupling risk:** an existing add-on pack
  (MarcelRoozekrans/superpowers-extensions) broke silently when v7 changed
  superpowers' wording. spx therefore lists every upstream touchpoint it
  relies on (8.2) and tests them against each new superpowers release.

## 7. Design

### 7.1 How spx hooks into superpowers

spx uses two portable mechanisms.

1. **An spx block in the repo's instructions files.** This is the backbone,
   because superpowers lets project instructions override its defaults.
   `spx:setup` writes the block between markers that carry spx's version:

   ```markdown
   <!-- spx:begin v1.0.0 -->
   ...rules...
   <!-- spx:end -->
   ```

   Update mode replaces everything between the markers.
2. **Skill descriptions that name superpowers' moments,** for example "Use
   when brainstorming saves or revises a spec…" or "Use after writing-plans
   saves a plan…".

The skills themselves carry the full rules. The block reinforces when they
fire, and adds the few rules other skills must follow. The block must say,
in substance (the builder chooses the wording and tests it):

- **Lifecycle:** specs follow spx's format and lifecycle. Use
  `spx:spec-lifecycle` whenever a spec is created, revised, sent for review,
  planned from, finished, split into further specs, or dropped.
- **Revising:** when brainstorming starts from an existing spec, revise that
  file in place. Keep spx's front matter, decisions list and owner review
  section, and add any new decisions to the list. Do not write a new dated
  file.
- **Plan overview:** after writing-plans saves a plan, and before asking for
  plan review, use `spx:illustrate-plan`.
- **Sidecars:** `.overview.md` files next to plans are for people.
  - Never plan or implement from them.
  - Only `spx:illustrate-plan` writes them.
  - Feedback on an overview is feedback on the plan: fix the plan (and the
    spec if needed) first, then regenerate the overview.
- **Ownership memory:** `docs/spx/ownership.md` is read only by spx skills,
  at the moments they need it.
- **Finishing:** if an open PR already exists for the spec, the plan, the
  overview and the code go into it. Do not open a second PR.

**Which instructions files.** Each harness reads its own:

- Claude Code: `CLAUDE.md`
- Copilot CLI: `AGENTS.md` and/or `.github/copilot-instructions.md`
- Kiro: `.kiro/steering/` and/or `AGENTS.md`

The builder must check which files each harness actually loads, and pick the
smallest set that reaches all three. Prefer one shared copy of the block,
with the other files pointing to it if the harness supports that. Otherwise
setup writes the block into each file.

**Sessions started above the repo.** A session started in a parent folder
that holds several repos may not load a repo's instructions files at all.
For that reason:

- the skill descriptions and skill bodies carry everything spx needs, so spx
  still works without the block, only with less reliable triggering
- spx's README recommends starting sessions inside the repo *(my call)*

### 7.2 Skills

| Skill | Fires | Purpose |
|---|---|---|
| `spx:setup` | Only when someone asks for it | Prepare, review or update a repo for spx, including the ownership memory |
| `spx:spec-lifecycle` | On its own, at spec moments | Spec format, related specs, status changes, spec PR reviewers, archiving; ships the checker |
| `spx:illustrate-plan` | On its own after each saved plan, and on request | Write the plan's overview sidecar |

- **No separate ownership skill.** "Review code ownership" means running
  `spx:setup` in update mode with the ownership step selected. *(my call)*
- **No plugin agents in v1.** When a skill needs a helper, it hands a generic
  subagent a prompt file shipped with the skill, the way superpowers does.
  Agent definition formats differ from harness to harness. *(my call)*

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
- **Refreshing:** only through `spx:setup` (update mode, ownership step).
  spx never checks or refreshes the memory on its own.
- **Readers:**
  - `spx:spec-lifecycle`, for owner sections and reviewers
  - `spx:illustrate-plan`, to tag each task with its owner area
  - `spx:setup`, which builds and reviews it

  Nothing else reads it.

### 7.4 Spec format

spx adds to superpowers' spec. It never restructures it. Superpowers'
content stays exactly as brainstorming writes it, and spx adds three parts:

```markdown
---
status: assessed
assessed-at: 2026-10-17T14:05:00Z
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
| `assessed-at` | Once the spec has been assessed | UTC timestamp (ISO 8601) of when it became `assessed`. Reset on every revision while it is still `assessed`. Drives the 14-day flag (7.5) |
| `relates-to` | Optional | Repo-relative paths to related specs |

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
  `spx:spec-lifecycle` judges which ones the spec affects, from the paths and
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

**Location:**

- Specs live wherever superpowers saves specs in that repo (default
  `docs/superpowers/specs/`).
- Specs that are `done` or `abandoned` move to an `archive/` subfolder there.
- Specs that are `draft`, `assessed` or `in-progress` never move.

### 7.5 Lifecycle

There are five statuses. There is no "outdated": a stale spec goes back to
`draft`.

| Moment | What spx does | Where the change lives |
|---|---|---|
| Brainstorming saves a new spec | `status: draft` | Merged to main on its own, through the team's flow |
| The dev says the revised spec is ready for review | `status: assessed`, `assessed-at` = now; spx opens the spec PR | Feature branch / PR |
| The spec is revised while `assessed` (e.g. after review comments) | `assessed-at` reset to now; reviewers added for any newly affected area | Same PR |
| "Ready for review" again while the PR is open | Update the existing PR; never open a new one | Same PR |
| writing-plans saves a plan from the spec | `status: in-progress`; confirm the plan's `**Spec:**` line; run `illustrate-plan`; commit and push the plan and overview to the PR before any code | Same PR |
| The finishing step | `status: done` (see "Several plans" below); spec moved to `archive/`; links fixed | Same PR, before merge |
| Someone says the feature is dropped | `status: abandoned`; spec moved to `archive/`; links fixed | Wherever the spec is |

**Where statuses show up.** Main only ever shows `draft`, `done` or
`abandoned`. `assessed` and `in-progress` exist on the feature branch.

**Every status change is committed.**

**The 14-day rule.**

- An `assessed` spec is trusted for 14 days from `assessed-at`.
- Whenever `spx:spec-lifecycle` meets an `assessed` spec past that point
  (notably when planning starts from it), it warns that the assessment may
  no longer hold and sets the spec back to `draft`.
- It doesn't block. If the dev carries on and a plan is saved, the status
  becomes `in-progress` as usual.

**Opening the spec PR.**

- **Status:** set `assessed` and `assessed-at`, then commit.
- **Branch:** push the current branch. If the dev is on the default branch,
  ask for a branch name first, since teams name branches after JIRA keys.
  *(my call)*
- **PR:** open it with `gh` and fill in the repo's PR template.
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
- **Worktrees:** if superpowers created a separate worktree branch for
  execution, finishing merges it back into the PR's branch and pushes.
  finishing-a-development-branch already merges back into the branch the work
  came from, so the block only has to say that this is the PR's branch.
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
  so nobody forgets to close it. *(my call)*
- **Moves:** when a spec moves to `archive/`, spx updates every `relates-to`
  entry and `**Spec:**` line that points to it.

### 7.7 Plan overview sidecar (`spx:illustrate-plan`)

- **Why:** plans are long, detailed and technical. People review the overview
  instead: first the dev, then one or more of the spec's reviewers, in the
  feature PR, before any code is pushed.
- **When:**
  - on its own after writing-plans saves a plan, before the plan review
    question *(my call: automatically after every plan)*
  - whenever someone asks for it, optionally naming what to show
- **Where:** next to the plan, with the same name plus `.overview`:

  ```
  docs/superpowers/plans/2026-10-17-checkout-redesign.md            ← plan (for agents)
  docs/superpowers/plans/2026-10-17-checkout-redesign.overview.md   ← sidecar (for people)
  ```

  Plans and sidecars never move. Only the spec moves to `archive/`.
- **First line:** it names the file for what it is:
  *"Overview for people of `<plan path>`. Not a plan; never plan or
  implement from it."*
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
- **Keeping it in step:** when the plan changes, the sidecar is regenerated.
  Nobody hand-edits a sidecar to fix a plan problem.

### 7.8 Checker

- **What it is:** a script shipped inside `spx:spec-lifecycle`.
  - Plain Node, built-in modules only (see Constraints).
  - It only reports. It never edits files and never stops anything.
- **What it checks:**

  | Area | Check |
  |---|---|
  | Front matter | Present and within the allowed format; inline lists flagged |
  | Front matter | `status` is one of the five values |
  | Front matter | `assessed-at` is a valid timestamp, and present when `status: assessed` |
  | Staleness | An `assessed` spec past 14 days is reported as stale |
  | Links | Every `relates-to` entry points to an existing spec |
  | Sections | `## Decisions` and `## Owner review` are present |
  | Location | `done` and `abandoned` specs are in `archive/`; other specs are not |
  | Branch | On the default branch, only `draft`, `done` or `abandoned` (only when run inside git) |
  | Hint | A `draft` spec whose related specs are all `done` or `abandoned` |
  | Plans | Every plan's `**Spec:**` line points to an existing spec |
  | Plans | Every plan has its `.overview.md` sidecar |
  | Sidecars | The first line names its plan |
  | Sidecars | The sidecar is not older than its plan (by last commit, else by file time) |

- **Older files:** specs and plans from before spx was set up in the repo
  must not be flagged. How setup marks that point is left to the builder.
- **Output:** a short report to read, with errors and warnings.
  - The exit code is non-zero only on errors, so CI can show a failing check.
  - Whether that failure blocks a merge is the team's branch-protection
    choice, not spx's.
- **When the skills run it:**
  - after a spec is saved or revised
  - before opening or updating the spec PR
  - after the plan and overview are saved
  - at finishing, after archive moves

  The skills show the dev the report and carry on.
- **In CI:** setup can add a workflow that runs it on PRs (7.9, step 7).

### 7.9 Setup (`spx:setup`)

- **It runs only when someone asks,** and it goes through numbered steps.
- **When called, it asks two things:**
  - **Mode:** `init` by default if the repo has no spx setup yet (no spx
    block in the instructions files and no `docs/spx/ownership.md`).
    Otherwise `update/review` by default.
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
4. **Instructions files:** write or replace the versioned spx block in every
   instructions file the target harnesses read (7.1).
5. **PR template:** check that one exists, and offer a starter for spec PRs
   if it doesn't.
6. **Branch protection:** check whether code-owner approval is required.
   Report and recommend only. Setup never changes repo settings. *(my call)*
7. **CI check (optional):** offer a GitHub Actions workflow that runs the
   checker on PRs.
   - The repo needs its own copy of the checker for this, stamped with spx's
     version.
   - Update mode refreshes that copy.
   - Where the copy lives is left to the builder; `docs/spx/` is suggested.
8. **Summary and commit:** show every change, then commit after the user
   confirms.

**Update/review mode.** Each step compares what's in the repo with what spx
expects, reports the differences, and proposes fixes. It doesn't rebuild
from scratch. Re-running it after an spx upgrade refreshes the block and the
checker copy.

### 7.10 Team setup list (for spx's README)

To get the most out of spx, a team needs:

1. superpowers and spx installed in every tool the team uses
2. CODEOWNERS using GitHub accounts, with comments describing what each area
   covers
3. a PR template for spec PRs
4. the GitHub CLI installed and signed in wherever spx runs, and Node for the
   checker
5. a branch protection rule requiring code-owner approval, which is where the
   process gets enforced
6. optionally, the checker running in CI
7. `spx:setup` run once in init mode, then re-run in update mode when
   ownership changes or spx is upgraded
8. sessions started inside the repo, not in a parent folder

## 8. Packaging and upstream

### 8.1 Packaging

- **Repo and manifest:** one repo, `spx`, with a Claude-style plugin manifest
  and marketplace. Copilot CLI installs Claude-style marketplaces directly.
- **Kiro:**
  - The team installs superpowers in Kiro itself.
  - The builder works out how Kiro installs spx's skills. Superpowers' open
    PR #2126 (Kiro through a native custom agent and Agent Skills) is a
    reference.
  - If Kiro can load skills only through an agent file, spx must not edit
    superpowers' agent file. Instead, spx's install notes tell the team how
    to add spx's skills to their own Kiro setup.
- **Superpowers version:**
  - spx declares superpowers as a dependency with a *tested version range*,
    in harnesses that support it (Claude Code's `plugin.json`
    `dependencies`). Elsewhere it documents the range as a prerequisite.
  - It is a range, not an exact version, so teams still get superpowers'
    fixes.

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
  the work came from, and uses the repo's PR template.

spx's own CI runs a contract test against superpowers' latest release. The
test fails when any of these disappears or changes, so the break is caught
before teams hit it.

## 9. Acceptance

spx's skills shape agent behavior, so they are tested as behavior, in each
of the three harnesses. Each test uses ordinary prompts and never names an
spx skill, except `spx:setup` and on-request `illustrate-plan`.

In Kiro, the tests count only once superpowers' own check passes there:
"Let's make a react todo list" must trigger brainstorming.

1. **New spec:** brainstorming a new feature produces a spec with
   `status: draft`, a decisions list, and an owner review section that
   matches the ownership memory.
2. **Revision:** brainstorming with an existing draft as input updates that
   file in place, adds decisions, and creates no second spec.
3. **Ready for review:** saying "the spec is ready for review" sets
   `assessed` and `assessed-at`, and opens a PR whose reviewers are the
   affected areas' owners. Saying it again updates the same PR.
4. **Revision under review:** revising an `assessed` spec resets
   `assessed-at` and adds reviewers for newly affected areas.
5. **Stale spec:** planning from an `assessed` spec more than 14 days past
   `assessed-at` warns and resets it to `draft`, without blocking the plan.
6. **Plan saved:** the spec becomes `in-progress`, and an `.overview.md`
   sidecar with the default contents appears before the plan review
   question. Both are pushed to the PR.
7. **Finishing:** the spec becomes `done` and moves to `archive/`. Its
   `relates-to` and `**Spec:**` links are fixed, and the work goes into the
   existing PR. No new PR is opened, including when a worktree was used.
8. **Splitting a super-spec:** creating a spec from a draft super-spec links
   the two in both directions, and the super-spec's status stays unchanged.
9. **Checker:** the checker reports each check in 7.8 on purpose-built bad
   fixtures, passes clean ones, ignores older files, and runs with no
   installed packages.
10. **Setup:** `init` on a bare repo, `update/review` on a set-up repo, and
    step selection all behave as in 7.9.
11. **Sidecars:** asked to implement the plan, the agent works from the plan
    and the spec, not the sidecar.

The builder may use superpowers' eval harness (superpowers-evals) or plain
scripted sessions. The checker and the contract test are ordinary automated
tests.

## 10. Risks

- **Brainstorming's "invoke writing-plans and no other skill"** may keep
  `spx:spec-lifecycle` from running when a spec is saved. The instructions
  block mitigates this, scenario 1 must prove it works, and the checker
  reports a spec with no front matter if it slips through.
- **Brainstorming can ignore instructions about where outputs go** (open
  issue #939). Scenario 2 must prove in-place revision. The checker can't
  reliably detect a duplicate spec, so this risk stays partly open.
- **Upstream changes:** caught by the contract test (8.2).
- **Unverified in Copilot CLI and Kiro:** how skill descriptions surface,
  and which instructions files load.

## 11. Left to the builder

- **Wording:** the exact wording of the skill descriptions and the
  instructions block. Develop it test-first with
  `superpowers:writing-skills`.
- **Formats:** the `ownership.md` layout and the Mermaid details.
- **Setup detection:** how `spx:setup` detects an existing setup, and how it
  marks where spx adoption starts, so the checker ignores older files.
- **Harness specifics:** which instructions files to write per harness
  (7.1), and how Kiro installs the plugin (8.1).
- **Locations:** where the repo's copy of the checker lives (7.9).
- **Layout:** the internal file layout of the plugin and of each skill.
