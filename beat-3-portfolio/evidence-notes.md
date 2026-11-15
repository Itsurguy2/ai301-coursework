# Evidence notes for Assignment 6 (not graded, collected 2026-11-15)

Raw material collected from my record. The prose in `technical-writeup.md` and
`linkedin-post.md` is mine to write. These are only facts and excerpts to quote.

## Where things stand today (checked 2026-11-15)

- PR: https://github.com/LawnchairLauncher/lawnchair/pull/7477
  "feat(search): add a switch to turn off web search suggestions", into `16-dev`
- State: **OPEN**, opened 2026-11-02 05:00 UTC. No human review and no maintainer comment.
  Last activity 2026-11-02 (CodeRabbit's summary). That's **13 quiet days** as of today.
- Checks: CodeRabbit `pass`. GitHub: MERGEABLE, merge state BLOCKED (waiting on review).
- Issue #5490: open. My claim comment (2026-11-02) has no maintainer reply:
  https://github.com/LawnchairLauncher/lawnchair/issues/5490#issuecomment-6567947612
- My playbook nudges only after 21 quiet days, which is **2026-11-23**. The deadline is before that.
- Wording note for the LinkedIn post: "opened a fix", not "fixed". Also "awaiting review".

## Timeline (dates from contribution-story.md, gate-log, and monitor-log)

| Date | What happened |
|---|---|
| 2026-10-18 | issue-scout scan 1: 103 unique issues, then 18 after ranking, then 5 graded, **0 accepted**. All 5 had an open linked PR |
| 2026-10-18 | Added `-linked:pr` to every query. Screened 18 Issue Selection picks. Graded 4, all accepted, #5490 among them |
| 2026-11-01 | Re-graded after the unit-6 scope patch. #5490 still accepted, ranked first on fit, "moderate" difficulty |
| 2026-11-01 | Installed the ship-gate. Fixed manifest rows for `cmd.exe` (`.\gradlew.bat`, `JAVA_HOME` set per row) |
| 2026-11-01 | plan-check run 1 flagged two honesty gaps. Run 2 accepted |
| 2026-11-01 | Commits `b6ae3ea` and `e5d0aa0` on `feat/5490-web-search-suggestions-toggle`: 4 files, 16 added lines |
| 2026-11-01 23:55 | ship-gate manual run: PASS (3 repo checks + 8 pr-precheck checks), then a crash printing the summary (`UnicodeEncodeError`, cp1252 can't print `→`) |
| 2026-11-01 23:59 | Hook: "PASS (honored the manual pass at 2026-11-01 23:55)". PR opened |
| 2026-11-02 02:56 UTC (= 11-01 21:56 EST) | GitHub's `createdAt` for my claim comment on #5490. **Check this:** contribution-story says I claimed before setting up the JDK, so the comment may have been reposted, or the course's posting rules delayed it. Confirm the real claim date before putting it in the timeline |
| 2026-11-02 05:00 | PR #7477 opened (UTC). CodeRabbit: "No actionable comments", effort "2 (Simple)" |
| 2026-11-10 18:21 | pr-monitor `--state` mislabeled the PR **FEEDBACK-IN** (the bot summary counted as feedback) |
| 2026-11-10 18:23 | After the bot-comment playbook rule: full run drafted **no-send, wait note** |

Timezone note: gate-log times are local (EST). GitHub times are UTC. 23:55 EST on 11-01
is 04:55 UTC on 11-02, so the PR opened 4–5 minutes after the gate passed, which matches
your Unit 7 entry.

## Proof excerpts (from real work, ready to paste)

### 1. The bug, in the code (pinned commit ff03721)

One boolean gates both results:

```
PreferenceManager.kt:162:    val searchResultStartPageSuggestion = BoolPref("pref_searchResultStartPageSuggestion", true, recreate)
WebSuggestionProvider.kt:28:        if (query.isBlank() || !prefs.searchResultStartPageSuggestion.get()) {
LawnchairLocalSearchAlgorithm.kt:151:        if (prefs.searchResultStartPageSuggestion.get()) {
SearchProviderPreference.kt:182:                valueRange = 2..10,
```

### 2. The fix: the core hunk of the diff (WebSuggestionProvider.kt)

```diff
             return flow { emit(emptyList()) }
         }
 
+        if (!prefs2.showWebSearchSuggestions.firstCached()) {
+            return flow { emit(emptyList()) }
+        }
+
         val provider = prefs2.webSuggestionProvider.firstCached()
```

and the new preference (PreferenceManager2.kt):

```diff
+    val showWebSearchSuggestions = preference(
+        key = booleanPreferencesKey(name = "show_web_search_suggestions"),
+        defaultValue = true,
+    )
```

### 3. Before/after test table (same emulator, same `weather` query)

| Setting | Before (`ff03721`) | After (`e5d0aa0`) |
|---|---|---|
| Defaults | 3 suggestions + "Search on Google" | the same |
| Web suggestions on, new Search suggestions off | not possible | "Search on Google" only, no suggestion rows |
| Web suggestions off | nothing from the web | nothing from the web (unchanged) |

CI checks on the branch: `spotlessCheck` (8s), `assembleLawnWithQuickstepGithubDebug`
(6m 57s), and `assembleLawnWithQuickstepPlayDebug` (7m 37s), all BUILD SUCCESSFUL.

### 4. ship-gate log (gate-log.md, 2026-11-01)

```
## 2026-11-01 23:55 [manual] LawnchairLauncher/lawnchair @ feat/5490-web-search-suggestions-toggle (base 16-dev)
- verdict: PASS
  - diff-matches-plan [pass]: All 4 hunks (...) map 1:1 to plan items 1-4; no unplanned hunks
  - diff-free-of-debris [pass]: Diff is 4 small hunks, 16 lines, no debug output, dead code, or unrelated edits
## 2026-11-01 23:59 [hook] ...
- verdict: PASS (honored the manual pass at 2026-11-01 23:55)
```

### 5. PR monitor before and after the bot rule (monitor-log.md, 2026-11-10)

```
## 2026-11-10 18:21 [state]
- state: feedback-in: review feedback is in (latest from the thread: comment 8 days ago)

## 2026-11-10 18:23 [run]
- move: no-send — wait note (bot-only thread, nudge threshold unmet)
- mergeability: MERGEABLE/BLOCKED
```

Rule added to playbook.md:
> Bot comments (CodeRabbit summaries, or a "No actionable comments" review) get no reply.
> Only an actionable bot finding I actually fix gets a one-line "Fixed in <commit>".

Staleness threshold, measured from five real PRs (#7499, #7500, #7482, #7457, #7412):
0.8, 1.3, 3.0, 12.0, and 19.6 days to a first maintainer response, so the first nudge comes after **21 quiet days**.

## Candidates for "a change I made" (second versions, each with a cause)

1. **issue-scout `-linked:pr` fix.** Scan 1 graded 5 and accepted 0, because all 5 had
   open linked PRs (`--no-assignee` is useless in repos that never assign). Added
   `-- -linked:pr` to every query (it has to go last, or it swallows the other flags).
   Re-running returned none of the 5 rejects.
2. **plan-check made the plan honest.** Run 1 flagged that "stops the network call"
   was a claim nothing in my testing observed, and that "change the plan before opening a
   PR" was a promise I couldn't keep. Run 2 accepted, after I marked the network claim as
   coming from reading the code and changed the promise to "tell me and I'll change it".
3. **repro-check caught a trimmed grep.** I'd cut a three-line grep output to one line,
   and the dropped line made one sentence wrong. I pasted the full output and fixed the
   sentence.
4. **PR monitor bot rule.** See excerpt 5 above. A clean before and after, one run apart.
5. **ship-gate manifest for Windows.** Rows failed under `cmd.exe`. Each row now sets
   `JAVA_HOME` and calls `.\gradlew.bat`.

## Candidates for "a result that landed"

- The PR itself is **not** merged or reviewed. Say that plainly.
- Things that did happen because of the work:
  - The PR exists upstream, it passed Lawnchair's CI-equivalent checks plus CodeRabbit,
    and GitHub reports it MERGEABLE (only review is missing).
  - The ship-gate's manual pass was honored by the hook when the PR opened.
  - The PR monitor's next run after the bot rule drafted the right move (no-send),
    where the run before it had mislabeled the thread. ("Your tool's next run
    catching something it had missed" is one of the assignment's own examples.)
  - After `-linked:pr`, the scout's next screening led to 4 accepted issues, one of which
    became this PR.

## Honest limits (raw list)

- No maintainer answer yet on the design question (keep "Search on Google" when
  suggestions are off?), so the PR builds one reading of the issue, and they may redirect it.
- That the network request is skipped comes from reading the code. Traffic was never captured.
- Tested only on an emulator, only with Google as the provider, and not on physical phones.
- No unit test, because there's no Gradle unit-test source set for this code.
- First launch after clearing data froze once (ANR during startup, before search), and
  a cold start on the unmodified build wasn't tested for comparison.
- Tools: the scout is a good grader and a weak finder. It has no repo-level difficulty
  check and no size signal. repro-check's "Behavior matches the issue" is written for bugs,
  not feature requests. The ship-gate crashes on a cp1252 console.
- Lessons already written: set up the build before claiming, ask the design question as
  a yes/no question, and set `PYTHONIOENCODING=utf-8` up front.
