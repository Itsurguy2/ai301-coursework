# Splitting Lawnchair's "Web suggestions" switch (#5490), and teaching my PR monitor to ignore bots

## Overview

[Lawnchair](https://github.com/LawnchairLauncher/lawnchair) is an open source Android
home-screen app, also called a launcher. In its app drawer search, a single "Web
suggestions" switch controls two separate features: the autocomplete suggestions from the
search provider, and the "Search on Google" button. Turning it off removes both, so
there's no way to keep web search without the suggestions.
[Issue #5490](https://github.com/LawnchairLauncher/lawnchair/issues/5490) asks for that
option. My fix adds a separate "Search suggestions" switch that is on by default, so
nothing changes by default. The change is 16 added lines across 4
files. I also built a PR monitor that tracks the pull request after it opens, and wrote its
playbook, with two rules that matter here: wait 21 quiet days before
nudging the maintainers, and don't count bot comments as feedback.
[PR #7477](https://github.com/LawnchairLauncher/lawnchair/pull/7477) is open and waiting
for its first review from a maintainer.

## What happened

### 1. Finding the issue

I built a scout that searches GitHub and grades issues against a rubric. Its first scan
graded 5 issues and accepted none, because every one already had a pull request linked to
it. I added `-linked:pr` to every query to skip those. I also stopped relying
on the scout's search and used it only as a grader on hand-picked issues. I screened 18
with quick `gh` checks, graded the 4 that were left, and #5490 ranked first on fit. The
scout rated it "moderate", because the issue names the setting's place in the app but not
in the code.

### 2. Reproducing it

I built commit `ff03721` on
`16-dev`, the branch Lawnchair takes pull requests on, and pinned it so my before and
after would compare the same code. On an Android 16 emulator with default settings, I
searched `weather` in the app drawer and got three Google suggestions and a "Search on
Google" button. With "Web suggestions" off, both disappeared together.

The code explained why. One setting is read in both places:

```
PreferenceManager.kt:162:    val searchResultStartPageSuggestion = BoolPref("pref_searchResultStartPageSuggestion", true, recreate)
WebSuggestionProvider.kt:28:        if (query.isBlank() || !prefs.searchResultStartPageSuggestion.get()) {
LawnchairLocalSearchAlgorithm.kt:151:        if (prefs.searchResultStartPageSuggestion.get()) {
```

Line 28 decides whether to fetch suggestions, and line 151 decides whether to add the
"Search on Google" button. Because both check the same value, the fix needed a second,
separate setting.

### 3. Planning

Lawnchair's [preferences docs](https://docs.lawnchair.app/developers/concepts/preferences)
call `PreferenceManager2` "the required location for all new settings", so the new setting
went there instead of the legacy `PreferenceManager`. A plan checker I wrote,
which compares each claim against the code and my evidence, caught one claim: I'd written that turning the option off "stops the network call", but nothing
in my testing would show that. I rewrote it to say the claim comes from reading the code,
and listed it as untested.

### 4. Building and testing

I made two commits. `b6ae3ea` adds the setting and the check, and `e5d0aa0` adds the
switch and two English strings. This is the core change in `WebSuggestionProvider.kt`:

```diff
@@ -29,6 +29,10 @@ object WebSuggestionProvider : SearchProvider {
             return flow { emit(emptyList()) }
         }

+        if (!prefs2.showWebSearchSuggestions.firstCached()) {
+            return flow { emit(emptyList()) }
+        }
+
         val provider = prefs2.webSuggestionProvider.firstCached()
```

There are no Gradle unit
tests for this code, so I re-ran the reproduction on both builds instead:

| Setting | Before (`ff03721`) | After (`e5d0aa0`) |
|---|---|---|
| Defaults | 3 suggestions + "Search on Google" | the same |
| Web suggestions on, Search suggestions off | not possible | "Search on Google" only |
| Web suggestions off | nothing from the web | nothing from the web |

The middle row proves the fix. The other two show nothing else changed.

### 5. Opening the PR

Before the PR opened, a gate I set up ran three of Lawnchair's own CI
checks (`spotlessCheck` and two debug builds) plus 8 of my own, such as whether the
description matches the diff. All passed, then the script crashed while
printing its summary, because the Windows console can't print `→`. Before trusting the
pass, I checked the gate's log and confirmed it had been recorded before the crash.

### 6. The monitor's mistake, and the rule that fixed it

To set the nudge threshold, I measured five recent merged PRs from outside contributors
(#7499, #7500, #7482, #7457, #7412). They waited 0.8, 1.3, 3.0, 12.0 and 19.6 days for a
first response, so the playbook waits 21 quiet days, just past the slowest.

On day 8, the monitor labeled my PR `FEEDBACK-IN`. The only "feedback" was CodeRabbit's
bot summary saying it had no actionable comments. I added a rule
to the playbook:

> **TODO: paste the bot-comment rule exactly as written in the playbook.**

With that rule, the next run drafted "no-send, keep waiting":

```
TODO: paste the 18:21 monitor-log.md entry (FEEDBACK-IN)
TODO: paste the 18:23 monitor-log.md entry (no-send, 8 of 21 days)
```

I posted nothing.

### 7. Where it stands

As of 2026-11-15, the PR has been quiet for 13 days. GitHub says it can be merged once
it's reviewed, and the playbook's first nudge is due on 2026-11-23.

## Limitations and what's next

- **The design question is still open.** I asked the maintainers which behavior they
  wanted and haven't had an answer, so the PR builds the reading the issue's wording
  supports. **Next:** ask it as a yes/no
  question: "Keep Search on Google when suggestions are off: yes/no?"
- **"The request is skipped" comes from reading the code.** I tested only on an emulator
  with Google, without capturing network traffic. **Next:** test on a physical
  phone and capture traffic with the switch off.
- **The monitor only recognizes bots through a rule I wrote by hand.** A new bot could
  trigger the same false `FEEDBACK-IN`. **Next:** have it check the commenter's account
  type (GitHub's API marks bots as `"type": "Bot"`) instead of the comment's wording.
