# Contribution: [FEATURE] Disabling search suggestions for web suggestions

Path: `beat-2-contribution/contribution-story.md`

This file is Assignment 5, and it is graded at the path above; a copy kept anywhere else in
the repository is not read. **Keep every section header below under its exact name** — the
format criteria grade exactly that. Your course repo link is the whole submission: no tool
files and no evidence uploads are part of Assignment 5.

**Contribution Number:** [1]
**Student:** Jesse Rosenthal
**GitHub Username:** Itsurguy2
**Issue:** https://github.com/LawnchairLauncher/lawnchair/issues/5490
**Pull Request:** [GitHub PR link, once opened]

One repository, built across units 5 to 8, telling the whole story of your real
contribution. Fill it as you work, not at the end — the sections below ask for evidence that
is far easier to write down as it happens than to reconstruct in the last weekend.

Contribution Number stays `[1]` unless you make more than one contribution this term; it
just numbers them.

---

## Toolkit

Every story section below, from Why I Chose This Issue through Pull
Request, ends with a Toolkit block: which tool did that phase's work
and how you used it, what it produced, what difficulties or
shortcomings it presented, and what you changed or would change
because of them. Together the blocks are the thread the rubric's
Toolkit points read, and they are the raw material for your unit-9
portfolio, so keep them current as you work rather than
reconstructing them later.

A Toolkit block looks like this (copy it into each story section and
fill it there):

```text
**Toolkit**

- Tool: [which tool did this phase's work, and how you used it]
- What it produced: [described in a sentence or a short excerpt]
- Difficulties: [what fought you or fell short, or "none met"]
- What changed: [the patch, revision, or workaround, what you would
  update, or "nothing"]
```

This section also keeps one short dated entry per week of tool work,
starting below: what you built or changed that week, what broke, and
what you changed because of it. The blocks tell the story where it
happened; these entries are the toolkit's own timeline.

All four entries are graded. Keep every heading below, and fill each
one in the week it belongs to.

**Unit 5: Built issue-scout, a gh-search funnel that ranks for free and grades only the top five**

I turned my unit-1 grader into a scout that runs six `gh search issues`
queries traced to my fit profile, ranks the results by label, comment
count, and age, and grades the top five with my rubric. The first scan
graded five issues and accepted none: every one already had an open
linked PR, because `--no-assignee` means nothing in repos that never
assign issues. I added `-linked:pr` to every query and widened the
"Clear acceptance criteria" check to read comment threads after it
failed an issue whose maintainer had written the exact spec in a
comment; screening 18 Issue Selection site picks with the same free
checks left four that the scout then accepted.

**Unit 6: Installed the voice guard over one merged voice guide, and wrote a CLAUDE.local.md for Lawnchair**

I merged my three voice-guide copies (the pr-precheck copy was the
newest, so it became the one guide in `~/.claude/voice-guard/`), added
upstream rules for claims, questions, and pings, and hooked the guard
into Claude Code. It stopped the `gh issue comment --help` wiring test,
then passed my claim both in the manual check and in the hook on the way
out, and `guard-log.md` has all three lines. What broke: Git Bash on
Windows can't make real symlinks (`ln -s` failed with "Operation not
permitted"), so each skill's `voice-guide.md` is now a one-line pointer
to the canonical guide instead of a link. The repo memory file
records the one preference that gates both web behaviors, the files
around it, `16-dev`, and the commit format. I wrote it from the code I'd
already traced for the repro instead of running a separate orientation
agent, and kept it out of git with `.git/info/exclude`.

**Unit 7: [one line on the tool work]**

[2-4 sentences: the ship-gate and your gate manifest. What it produced
on your real branch, what fought you, what changed because of it.]

**Unit 8: [one line on the tool work]**

[2-4 sentences: the PR-monitor and your playbook. What it read, what it
drafted, what you did with the draft.]

---

## Why I Chose This Issue

My scout's own search across GitHub didn't produce this issue. Its
first scan graded five issues from big repos and rejected all five,
because each already had an open linked PR. So I switched to using the
scout as a grader for picks from the Issue Selection site. I screened 18
of them with free `gh` checks and graded four: ICTU/quality-time #12823
and #11842, and Lawnchair #5913 and #5490. All four were accepted on
2026-10-18. When I re-graded on 2026-11-01 after this week's scope
patch, #12823 was already gone (someone claimed it on 2026-10-22 and
opened PR #14940), so I re-graded the other three. All three were still
accepted. The scout ranked #5490 first on fit, with a **moderate**
difficulty note: the issue names the settings screen but not the code,
so finding where the change goes was up to me.

I chose #5490 because it's the kind of work my profile asks for: a
small, concrete settings toggle the team had already marked
`status: reviewed`, with nobody else in the thread. #11842 is Python,
my strongest language, but it's a whole new metric and collector with an
open design question (the scout called it **ambitious**), and #5913's
expected behavior is still undecided. Before claiming, I traced the code
and found that one preference, `pref_searchResultStartPageSuggestion`,
switches off both the "search the web" action and the autocomplete
suggestions. That made the issue concrete for me. I want to learn how a
large Kotlin/Android codebase handles preferences and its Compose
settings screens, and how to get a PR through a team that warns its
codebase "may not be suitable for beginners." An honest read of the
scout: it's a good grader and a weak finder. Its free checks (`-linked:pr`,
claim comments) did most of the useful work. Its verdicts held up: it
correctly accepted #5490, and rejecting #12823 by hand this week
matched what the rubric would have done. Its gaps are real, though. It
has no repo-level difficulty check (Lawnchair's beginner warning didn't
show up anywhere), and before this week it had no size signal at all.

Claim comment: https://github.com/LawnchairLauncher/lawnchair/issues/5490#issuecomment-6567947612

**Toolkit**

- Tool: issue-scout, my search strategy and rubric, used in direct
  mode on hand-picked URLs: `issue-scout: grade <url> <url> <url>`
- What it produced: accept on #5490 in both grade runs (2026-10-18 and
  2026-11-01), every required check passing, ranked first on fit, with
  difficulty note "moderate: the issue names the settings location but
  not the code location"
- Difficulties: the wild search itself surfaced no claimable issue
  (all five scan-1 picks had open PRs). The rubric has no repo-level
  difficulty signal (Lawnchair's CONTRIBUTING.md beginner warning), and
  before this week it had no size signal, so the big #11842 earned as
  many preferred passes as the small toggles.
- What changed: applied the unit-6 scope patch. "Scope fits a newcomer"
  became "scope-feasible", which only fails when there's no single
  piece of work or a maintainer has closed the door, and open edges now
  produce a straightforward/moderate/ambitious difficulty note. I kept
  my earlier upstream/wontfix trigger inside the new row and logged the
  swap in `scan-notes.md` on 2026-11-01.

---

## Understanding the Issue

**Problem Description**

[In your own words, what is broken or missing?]

**Expected Behavior**

[What should happen?]

**Current Behavior**

[What actually happens?]

**Affected Components**

[Which parts of the codebase are involved?]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Reproduction Process

**Environment Setup**

Windows 11 Home (10.0.26200), Git Bash, git, and the GitHub CLI. Project:
LawnchairLauncher/lawnchair on `16-dev`, the branch the developer docs
say PRs must target, pinned to commit
`ff037218d4769e362d77b3d7a8db01d63b4eead4` (2026-11-02). I forked it to
Itsurguy2/lawnchair and cloned with `git clone --recursive` (1.4 GB,
including the `platform_frameworks_libs_systemui` submodule), with
`upstream` set to the Lawnchair repo. To build, the docs ask for Android
Studio Preview and the `lawnWithQuickstepGithubDebug` variant, and CI
uses JDK 21. The challenge: this machine has no Android SDK, JDK, or
device yet, and the docs recommend macOS or Linux. So this first
reproduction reads the code at a pinned commit, and the on-device check
is still to do (see My findings).

**Steps to Reproduce**

1. `git clone --depth 1 -b 16-dev https://github.com/LawnchairLauncher/lawnchair.git && cd lawnchair`,
   then `git checkout ff037218d4769e362d77b3d7a8db01d63b4eead4` to pin it.
2. Find the string for the "Web suggestions" switch (Settings → Search bar → App drawer):
   `grep -n 'search_pref_result_web_title' lawnchair/res/values/strings.xml`
3. Find every place that reads the preference behind that switch:
   `grep -rn 'searchResultStartPageSuggestion' lawnchair/src --include=*.kt`
4. Check the sliders on the Web suggestions page:
   `grep -n 'valueRange' lawnchair/src/app/lawnchair/ui/preferences/components/search/SearchProviderPreference.kt`
5. Observed result: one boolean, `pref_searchResultStartPageSuggestion`,
   controls both the "search the web for <query>" action
   (`LawnchairLocalSearchAlgorithm.kt:151`) and the autocomplete
   suggestions fetched from the provider (`WebSuggestionProvider.kt:28`).
   Both UI switches write that same boolean, and the suggestion-count
   slider only goes from 2 to 10. So there's no way to keep web search
   but turn off its suggestions, which is the missing feature #5490 asks for.

**Reproduction Evidence**

- **Commit showing reproduction:** none. This is a code-level
  reproduction at upstream commit
  [ff03721](https://github.com/LawnchairLauncher/lawnchair/commit/ff037218d4769e362d77b3d7a8db01d63b4eead4),
  with no commit of my own yet.
- **Screenshots/logs:** output of steps 2-4 at that commit:

  ```
  1021:    <string name="search_pref_result_web_title">Web suggestions</string>

  lawnchair/src/app/lawnchair/preferences/PreferenceManager.kt:162:    val searchResultStartPageSuggestion = BoolPref("pref_searchResultStartPageSuggestion", true, recreate)
  lawnchair/src/app/lawnchair/qsb/providers/StartpageEU.kt:21:        val useWebSuggestions = prefs.searchResultStartPageSuggestion.get()
  lawnchair/src/app/lawnchair/search/algorithms/engine/provider/web/WebSuggestionProvider.kt:28:        if (query.isBlank() || !prefs.searchResultStartPageSuggestion.get()) {
  lawnchair/src/app/lawnchair/search/algorithms/LawnchairLocalSearchAlgorithm.kt:151:        if (prefs.searchResultStartPageSuggestion.get()) {
  lawnchair/src/app/lawnchair/ui/preferences/components/search/DrawerSearchPreferences.kt:160:        adapter = prefs.searchResultStartPageSuggestion.getAdapter(),
  lawnchair/src/app/lawnchair/ui/preferences/components/search/SearchProviderPreference.kt:152:            SearchProviderId.WEB -> it.searchResultStartPageSuggestion

  116:                valueRange = 2..10,
  182:                valueRange = 2..10,
  200:                        valueRange = 500..5000,
  ```

  (116 is the contacts slider, 182 is "Maximum number of suggestions",
  and 200 is the suggestion delay in ms.) Permalinks:
  [LawnchairLocalSearchAlgorithm.kt#L151](https://github.com/LawnchairLauncher/lawnchair/blob/ff037218d4769e362d77b3d7a8db01d63b4eead4/lawnchair/src/app/lawnchair/search/algorithms/LawnchairLocalSearchAlgorithm.kt#L151),
  [WebSuggestionProvider.kt#L28](https://github.com/LawnchairLauncher/lawnchair/blob/ff037218d4769e362d77b3d7a8db01d63b4eead4/lawnchair/src/app/lawnchair/search/algorithms/engine/provider/web/WebSuggestionProvider.kt#L28),
  [SearchProviderPreference.kt#L182](https://github.com/LawnchairLauncher/lawnchair/blob/ff037218d4769e362d77b3d7a8db01d63b4eead4/lawnchair/src/app/lawnchair/ui/preferences/components/search/SearchProviderPreference.kt#L182)
- **My findings:** the feature is still missing on `16-dev` today, and
  the cause is one preference doing two jobs. `StartpageEU.kt:21`
  reads the same preference, so the fix has to decide whether that
  provider follows the new toggle too. I haven't watched this happen
  in the app yet. Building `lawnWithQuickstepGithubDebug` and checking
  it on an emulator is my next step. I also asked the maintainers in my
  claim whether the new option should only turn off the autocomplete
  suggestions and keep the web search action.

**Toolkit**

- Tool: repro-check with the new wild scope
  (`repro-check-scope-wild.md`), run on my claim alone and then on my
  claim plus repro report together
- What it produced: accept both times. On the combined package it
  re-ran my greps at the pinned commit, confirmed all six
  `searchResultStartPageSuggestion` hits were complete, and passed every
  check, including "Honest outcome", because the report says it read
  the code rather than ran the app.
- Difficulties: it caught a real problem anyway. My Step 4 output block
  showed one `valueRange` line when the command prints three, and the
  line I left out (the delay slider) made my "only other control"
  sentence wrong. A trimmed excerpt labeled as command output is still
  not what the command printed.
- What changed: I pasted the full three-line output, said what each
  line is, narrowed the sentence to "the only count control", and moved
  the code-reading-not-device note to the top of the report.

---

## Solution Approach

**Analysis**

[Your analysis of the root cause: what is causing the issue, traced
to the files and functions involved.]

**Proposed Solution**

[High-level description of your fix approach, and what you are
deliberately NOT changing.]

**Implementation Plan**

1. [Modify file X to do Y]
2. [Add or change Z]
3. [Update tests]

[Record deviations here as they happen: what changed mid-build and
why. A recorded deviation is honest work; silent drift is not.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Testing Strategy

[What observable checks prove the fix: your reproduction steps re-run
against the built change (before and after), and the repo's own test
suite or checks with the outcome shown. "Tests pass" with nothing
observable named is not a strategy.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Implementation Notes

[A running log, added to as you work. Date each entry. What you
built, decisions you made when you made them, wrong turns included.]

**[Date]**

[Entry]

**[Date]**

[Entry]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Pull Request

**PR Link:** [GitHub PR URL when opened]

**PR Description:** [what the PR says; much of the content above
adapts into it]

**Maintainer Feedback:**

- [Date]: [feedback received, or "none yet"; silence is a normal
  state and costs you nothing]
- [Date]: [how you addressed it]

**Status:** [Awaiting review / Iterating / Merged / Closed. The
status is a fact you record, never a grade: the PR is graded as
opened.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Learnings and Reflections

**Technical Skills Gained**

[What you learned technically, named concretely.]

**Challenges Overcome**

[What was hard and how you solved it; point at the real events, the
Toolkit blocks above are full of them.]

**What I'd Do Differently Next Time**

[Reflection on your process.]

---

## Resources Used

[What actually helped, not a reading list. List things this story
already shows you using: the doc you followed to set the environment
up, the thread that explained the bug, the person who answered you.
One real entry beats ten plausible ones.]

- [Link to documentation that helped]
- [Discussion, issue, or post that helped]
