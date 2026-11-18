# Toolkit Portfolio: small tools that check my own work, from finding an issue to following up on the PR

**Student:** Jesse Rosenthal · **GitHub:** Itsurguy2
**Contribution:** [Lawnchair #5490](https://github.com/LawnchairLauncher/lawnchair/issues/5490) → [PR #7477](https://github.com/LawnchairLauncher/lawnchair/pull/7477) (open, awaiting first maintainer review)

---

## Featured Tools

**Why these tools:** Each phase of a contribution has a way to go wrong without noticing: picking an issue someone already took, posting a repro I didn't watch, promising something in a plan I can't back up, opening a PR that doesn't match its description, or nudging maintainers too early. I built one small tool for each of those mistakes, so every step gets checked before it reaches a real project.

| Phase | Tool | What it checks |
|---|---|---|
| Find an issue | issue-scout | Searches GitHub and grades issues against my rubric |
| Reproduce | repro-check | Checks my claim and repro against the issue and the code |
| Plan | plan-check | Checks each claim in my plan against the code and my evidence |
| Open the PR | ship-gate (runs pr-precheck) | Runs the repo's CI checks plus 8 of my own before a PR can open |
| After the PR | PR monitor | Sorts the PR into a state and drafts my next move, including "send nothing" |

---

### ⭐ Featured: the PR monitor

**What it does:** After a PR opens, it checks the thread, labels the PR's state, and drafts my next move from a written playbook. One possible move is "don't send anything."

**The setup: a wait time measured, not guessed.** I didn't want to nudge maintainers too early, so I measured five recent Lawnchair PRs from outside contributors (#7499, #7500, #7482, #7457, #7412). They waited **0.8, 1.3, 3.0, 12.0 and 19.6 days** for a first response. My playbook waits **21 quiet days** before a nudge, just past the slowest one.

**The friction:** On day 8, the monitor labeled my PR `feedback-in`, as if a reviewer had responded. The only comment was from CodeRabbit, a code-review bot, saying "No actionable comments." A person would have ignored it, but the monitor counted it as feedback.

**The change:** I added this rule to the playbook:

> Bot comments (CodeRabbit summaries, or a "No actionable comments" review) get no reply. Only an actionable bot finding I actually fix gets a one-line "Fixed in <commit>".

**The result:** The next run, two minutes later, recognized the bot-only thread and drafted "no-send, keep waiting." I posted nothing.

**Evidence** (`~/.claude/pr-monitor/monitor-log.md`, 2026-11-10):

```
## 2026-11-10 18:21 [state] https://github.com/LawnchairLauncher/lawnchair/pull/7477
- state: feedback-in: review feedback is in (latest from the thread: comment 8 days ago)
- move: none (--state only)

## 2026-11-10 18:23 [run] https://github.com/LawnchairLauncher/lawnchair/pull/7477
- state: feedback-in: review feedback is in (latest from the thread: comment 8 days ago)
- move: no-send — wait note (bot-only thread, nudge threshold unmet)
- mergeability: MERGEABLE/BLOCKED
```

**Honest limitation:** The fix is a hand-written rule that names CodeRabbit and its exact wording, so a different bot, or new wording, would trigger the same false `feedback-in`.

**What's next:** Have the monitor check the commenter's account type instead of the comment's text. GitHub's API marks bot accounts as `"type": "Bot"`, which would work for any bot.

---

### issue-scout

- **Friction:** The first scan graded 5 issues and accepted **none**, because every one already had a linked PR. I had filtered with `--no-assignee`, which does nothing in repos that never assign issues.
- **Change:** I added `-linked:pr` to every query and started using the scout as a grader on hand-picked issues, not as a finder.
- **Result:** Out of 18 screened issues, 4 were graded and all 4 accepted. #5490 ranked first on fit, rated "moderate" difficulty.
- **Limitation:** It's a good grader and a weak finder. It has no repo-level difficulty check (such as a CONTRIBUTING.md beginner warning).

### repro-check

- **Friction:** I had cut a three-line grep output down to one line, and the line I dropped made one of my sentences wrong. The tool caught it.
- **Change:** I pasted the full output, fixed the sentence, and replaced the code-only repro with a device run I actually watched.
- **Evidence:** the bug, in the code at pinned commit `ff03721`, where one setting controls both features:

```
PreferenceManager.kt:162:    val searchResultStartPageSuggestion = BoolPref("pref_searchResultStartPageSuggestion", true, recreate)
WebSuggestionProvider.kt:28:        if (query.isBlank() || !prefs.searchResultStartPageSuggestion.get()) {
LawnchairLocalSearchAlgorithm.kt:151:        if (prefs.searchResultStartPageSuggestion.get()) {
```

- **Limitation:** Its "Behavior matches the issue" check is written for bugs ("same error, same crash"), not feature requests.

### plan-check

- **Friction:** Run 1 caught two honesty gaps. I had claimed the fix "stops the network call," which nothing in my testing observed, and I had promised to change the plan "before opening a PR," which I couldn't keep with the maintainers silent.
- **Change:** I marked the network claim as coming from reading the code, and rewrote the promise as "tell me and I'll change it."
- **Result:** Run 2 accepted, with no open notes.

### ship-gate (with pr-precheck)

- **Friction:** On Windows, every check failed under `cmd.exe` until each row called `.\gradlew.bat` and set `JAVA_HOME` itself. Then, after a PASS, the script crashed printing its summary, because the console couldn't print `→`.
- **Change:** I checked `gate-log.md` before trusting the pass. It had been recorded before the crash.
- **Result:** It passed 3 Lawnchair CI checks plus all 8 pr-precheck checks, and the hook honored the pass when PR #7477 opened four minutes later.

```
## 2026-11-01 23:55 [manual] LawnchairLauncher/lawnchair @ feat/5490-web-search-suggestions-toggle (base 16-dev)
- verdict: PASS
  - diff-matches-plan [pass]: All 4 hunks (...) map 1:1 to plan items 1-4; no unplanned hunks
## 2026-11-01 23:59 [hook] ...
- verdict: PASS (honored the manual pass at 2026-11-01 23:55)
```

- **Next:** Set `PYTHONIOENCODING=utf-8` up front.

---

## The contribution these tools carried

**The bug:** In Lawnchair's app drawer search, one "Web suggestions" switch controls both the Google autocomplete suggestions and the "Search on Google" button. Turning it off removes both.

**The fix:** A new "Search suggestions" switch, on by default, so nothing changes for anyone who leaves it alone. The change is 16 added lines across 4 files.

```diff
+        if (!prefs2.showWebSearchSuggestions.firstCached()) {
+            return flow { emit(emptyList()) }
+        }
```

**Before and after** (same emulator, same `weather` search):

| Setting | Before (`ff03721`) | After (`e5d0aa0`) |
|---|---|---|
| Defaults | 3 suggestions + "Search on Google" | the same |
| Web suggestions on, new Search suggestions off | not possible | "Search on Google" only, no suggestions |
| Web suggestions off | nothing from the web | nothing from the web (unchanged) |

**Where it stands (2026-11-18):** PR #7477 is open and mergeable, with no maintainer review yet (16 quiet days). Following the playbook, the first nudge comes after 21 quiet days, on 2026-11-23.

---

## What's next

- **PR monitor:** detect bots by account type, not by comment wording.
- **The PR:** re-ask the open design question as a yes/no: "Keep Search on Google when suggestions are off: yes or no?"
- **Testing:** test on a physical phone and capture network traffic to confirm no suggestion request goes out when the switch is off.
