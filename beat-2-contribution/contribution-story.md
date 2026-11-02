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

Host: Windows 11 Home (10.0.26200) with Git Bash, git, and the GitHub
CLI. Project: LawnchairLauncher/lawnchair on `16-dev`, the branch the
developer docs say PRs must target, at commit
`ff037218d4769e362d77b3d7a8db01d63b4eead4` (2026-11-02). I forked it to
Itsurguy2/lawnchair and cloned with `git clone --recursive` (1.4 GB with
the `platform_frameworks_libs_systemui` submodule), with `upstream`
pointing at Lawnchair. Toolchain: Eclipse Temurin JDK 21.0.13 (CI uses
21), Android Studio Canary, and an Android SDK with platform
`android-37.2` and build-tools `37.0.0`, the versions `build.gradle` asks
for. `./gradlew assembleLawnWithQuickstepGithubDebug` built
`Lawnchair.16.Dev.(ff03721).github.debug.apk` in 12m 21s. I ran it on an
Android 16 emulator (Pixel 7 profile, Google APIs x86_64 image) with
Lawnchair set as the home app and every setting at its default.

Challenges: the machine started with no JDK or Android SDK, and the docs
recommend macOS or Linux. Three things fought me on Windows. First,
`sdkmanager.bat` splits package names at `;`, so I had to use the `/`
form (`platforms/android-37.2`). Second, `avdmanager` only accepts the
`;` form, so I ran it through `cmd` with the quotes kept. Third, my first
build failed with "The filename, directory name, or volume label syntax
is incorrect" because I'd written `local.properties` with single
backslashes, which the properties format reads as escapes. I rewrote
`sdk.dir` with forward slashes and it built. On the emulator, slow `adb`
swipes didn't open the app drawer, but a fast fling did.

**Steps to Reproduce**

1. Build and install the pinned commit: `./gradlew assembleLawnWithQuickstepGithubDebug`,
   then `adb install -r` the APK, and set Lawnchair as the home app.
2. Swipe up to open the app drawer, tap its search bar, and type `weather`.
   A "From the web" section shows three autocomplete suggestions from
   Google, with a "Search on Google" action below them.
3. Tap the gear, then Search bar → App drawer → "Web suggestions" (on,
   "Via Google"), and tap the label to open its page. It has one
   "Web suggestions" switch, "Maximum number of suggestions" (3), a
   delay slider (1000 ms), and the provider. There's no separate control
   for suggestions.
4. Drag "Maximum number of suggestions" all the way left. It stops at 2.
5. Turn the "Web suggestions" switch off, go home, open the drawer, and
   search `weather` again.
6. Observed result: nothing from the web. The autocomplete suggestions
   and the "Search on Google" action disappear together. There's no
   setting that keeps web search but drops its suggestions, which is
   the option #5490 asks for.

**Reproduction Evidence**

- **Commit showing reproduction:** none of my own. I built and ran
  upstream commit
  [ff03721](https://github.com/LawnchairLauncher/lawnchair/commit/ff037218d4769e362d77b3d7a8db01d63b4eead4)
  unchanged.
- **Screenshots/logs:** six emulator screenshots, kept locally with my
  repro report: suggestions on, the Search bar settings, the Web
  suggestions page, the slider at its floor of 2, the switch off, and
  the same search with nothing from the web. The installed build reports
  `versionName=16.Dev.(ff03721)`. The code behind it, at the same commit:

  ```
  lawnchair/src/app/lawnchair/preferences/PreferenceManager.kt:162:    val searchResultStartPageSuggestion = BoolPref("pref_searchResultStartPageSuggestion", true, recreate)
  lawnchair/src/app/lawnchair/qsb/providers/StartpageEU.kt:21:        val useWebSuggestions = prefs.searchResultStartPageSuggestion.get()
  lawnchair/src/app/lawnchair/search/algorithms/engine/provider/web/WebSuggestionProvider.kt:28:        if (query.isBlank() || !prefs.searchResultStartPageSuggestion.get()) {
  lawnchair/src/app/lawnchair/search/algorithms/LawnchairLocalSearchAlgorithm.kt:151:        if (prefs.searchResultStartPageSuggestion.get()) {
  lawnchair/src/app/lawnchair/ui/preferences/components/search/DrawerSearchPreferences.kt:160:        adapter = prefs.searchResultStartPageSuggestion.getAdapter(),
  lawnchair/src/app/lawnchair/ui/preferences/components/search/SearchProviderPreference.kt:152:            SearchProviderId.WEB -> it.searchResultStartPageSuggestion

  SearchProviderPreference.kt:182:                valueRange = 2..10,
  ```

  Permalinks:
  [LawnchairLocalSearchAlgorithm.kt#L151](https://github.com/LawnchairLauncher/lawnchair/blob/ff037218d4769e362d77b3d7a8db01d63b4eead4/lawnchair/src/app/lawnchair/search/algorithms/LawnchairLocalSearchAlgorithm.kt#L151),
  [WebSuggestionProvider.kt#L28](https://github.com/LawnchairLauncher/lawnchair/blob/ff037218d4769e362d77b3d7a8db01d63b4eead4/lawnchair/src/app/lawnchair/search/algorithms/engine/provider/web/WebSuggestionProvider.kt#L28),
  [SearchProviderPreference.kt#L182](https://github.com/LawnchairLauncher/lawnchair/blob/ff037218d4769e362d77b3d7a8db01d63b4eead4/lawnchair/src/app/lawnchair/ui/preferences/components/search/SearchProviderPreference.kt#L182)
- **My findings:** what I saw on the device matches the code. Both
  switches write one boolean, `pref_searchResultStartPageSuggestion`,
  and it gates both the "Search on Google" action (line 151) and the
  network autocomplete (line 28). That's why they disappeared together.
  The count slider's `2..10` range is the floor I hit. `StartpageEU.kt:21`
  reads the same boolean, so the fix has to decide whether that provider
  follows the new option too. I've only tested on an emulator with
  Google as the provider. My claim asked the maintainers whether the
  new option should turn off only the suggestions and keep web search,
  and there's no answer yet.

**Toolkit**

- Tool: repro-check with the new wild scope
  (`repro-check-scope-wild.md`), run three times: on my claim alone, on
  claim plus a code-only repro, then on claim plus the device repro
- What it produced: accept all three times. On the final package it
  re-checked every file:line at the pinned commit, confirmed it is the
  head of `16-dev`, and compared each screenshot against the step it
  illustrates.
- Difficulties: on the code-only run it caught that I'd trimmed a
  three-line grep output to one line, and the line I dropped made one
  of my sentences wrong. On the last run it pointed out that its
  "Behavior matches the issue" check is written for bugs ("same error,
  same crash"), so it had to read a feature request's "symptom" as the
  missing option.
- What changed: I pasted the full grep output and fixed the sentence.
  Then I replaced the code-only caveat with the device run instead of
  posting a repro I hadn't watched. I'd patch the rubric so "Behavior
  matches the issue" also covers feature requests (the gap the request
  names, at the place it names).

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
