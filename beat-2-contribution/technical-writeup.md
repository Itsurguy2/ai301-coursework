# Splitting Lawnchair's "Web suggestions" switch (#5490), and teaching my PR monitor to ignore bots

## Overview

[Lawnchair](https://github.com/LawnchairLauncher/lawnchair) is an open source Android
home-screen app, also called a launcher. The problem I worked on was in its app drawer
search. A single "Web suggestions" switch controls two separate features: the
autocomplete suggestions that come from the search provider, and the "Search on Google"
button. If you turn the switch off, you lose both, and there's no way to keep web search
without the suggestions. [Issue #5490](https://github.com/LawnchairLauncher/lawnchair/issues/5490)
asks for exactly that option. My fix adds a separate "Search suggestions" switch that is
on by default, so nothing changes for anyone who leaves it alone. The whole change is 16
added lines across 4 files. For my tooling, I built a PR monitor that tracks the pull
request after it opens. I wrote its playbook, and it has two rules that matter here: wait
21 quiet days before nudging the maintainers, and don't count bot comments as feedback.
Right now [PR #7477](https://github.com/LawnchairLauncher/lawnchair/pull/7477) is open
and waiting for its first review from a maintainer.

## What happened

### 1. Finding the issue

To find an issue, I built a scout that searches GitHub and grades what it finds against a
rubric. Its first scan didn't go well. It graded 5 issues and accepted none of them,
because every one already had someone's pull request linked to it. I had been filtering
with `--no-assignee`, but that does nothing in repos that never assign issues to anyone.
So I added `-linked:pr` to every search query, which skips issues that already have a PR.
I also changed how I used the scout. Instead of relying on its search, I hand-picked
issues and used it only as a grader. I screened 18 issues with quick `gh` checks, graded
the 4 that were left, and #5490 ranked first on fit. The scout rated it "moderate"
difficulty, because the issue says where the setting is in the app but not where it is
in the code.

### 2. Reproducing it

Before changing anything, I wanted to see the problem for myself. I built commit `ff03721`
on the `16-dev` branch, the branch Lawnchair takes pull requests on, and pinned that
commit so my "before" and "after" would compare the same code. I ran the build on an
Android 16 emulator using a Pixel 7 profile, left every setting at its default, and
searched `weather` in the app drawer. With "Web suggestions" on, I got three suggestions
from Google and a "Search on Google" button under them. Then I turned the switch off and
ran the same search. The suggestions and the "Search on Google" button both disappeared
at the same time. I also tried lowering "Maximum number of suggestions" to hide them, but
the slider stops at 2, so there was no setting that kept web search without the
suggestions.

The code explained what I saw on the screen. One setting,
`pref_searchResultStartPageSuggestion`, is read in both places:

```
PreferenceManager.kt:162:    val searchResultStartPageSuggestion = BoolPref("pref_searchResultStartPageSuggestion", true, recreate)
WebSuggestionProvider.kt:28:        if (query.isBlank() || !prefs.searchResultStartPageSuggestion.get()) {
LawnchairLocalSearchAlgorithm.kt:151:        if (prefs.searchResultStartPageSuggestion.get()) {
```

Line 162 defines the setting, and it's on by default. Line 28 decides whether to fetch
suggestions at all, and line 151 decides whether to add the "Search on Google" button.
Both check the same value, so one switch could never turn off only one of them. That
told me the fix needed a second, separate setting.

### 3. Planning

Lawnchair's [preferences docs](https://docs.lawnchair.app/developers/concepts/preferences)
say `PreferenceManager2` is "the required location for all new settings", so the new key
went there, not into the legacy `PreferenceManager`. My plan-check tool flagged one claim:
I'd written that turning the option off "stops the network call", but nothing in my
testing would observe that. I rewrote it to say the claim comes from reading the code, and
listed it as untested.

### 4. Building and testing

Two commits: `b6ae3ea` adds the preference and the check, and `e5d0aa0` adds the switch
and two English strings. The core change in `WebSuggestionProvider.kt`:

```diff
@@ -29,6 +29,10 @@ object WebSuggestionProvider : SearchProvider {
             return flow { emit(emptyList()) }
         }

+        if (!prefs2.showWebSearchSuggestions.firstCached()) {
+            return flow { emit(emptyList()) }
+        }
+
         val provider = prefs2.webSuggestionProvider.firstCached()
         val timeout = prefs2.maxWebSuggestionDelay.firstCached()
         val maxResults = prefs2.maxWebSuggestionResultCount.firstCached()
```

The check sits in its own `if`, before the provider is asked for anything, so the diff
shows the new check on its own lines. Lawnchair has no Gradle unit-test source set for
this code, so I couldn't write a unit test. Instead I re-ran the reproduction on both
builds, with app data cleared so every setting started at its default:

| Setting | Before (`ff03721`) | After (`e5d0aa0`) |
|---|---|---|
| Defaults | 3 suggestions + "Search on Google" | the same |
| Web suggestions on, Search suggestions off | not possible | "Search on Google" only |
| Web suggestions off | nothing from the web | nothing from the web |

The middle row proves the fix. The other two show nothing else changed.

### 5. Opening the PR

My ship-gate ran three of Lawnchair's CI checks (`spotlessCheck` and the GitHub and Play
debug builds) plus the 8 pr-precheck checks, and all passed. Then the script crashed while
printing its summary: the Windows `cp1252` console can't print `→`. Before trusting the
pass, I checked `gate-log.md` and confirmed the pass had been logged before the crash. The
hook accepted it when the PR opened four minutes later.

### 6. The monitor's mistake, and the rule that fixed it

To set the nudge threshold, I measured five recent merged PRs from outside contributors
(#7499, #7500, #7482, #7457, #7412). They waited 0.8, 1.3, 3.0, 12.0 and 19.6 days for a
maintainer's first response. So the playbook waits 21 quiet days, just past the slowest
one, before drafting a first-review nudge.

On day 8, the monitor labeled my PR `FEEDBACK-IN`. The only "feedback" was CodeRabbit's
bot summary: "No actionable comments were generated in the recent review." I added a rule
to the playbook:

> **TODO: paste the bot-comment rule exactly as written in the playbook.**

With that rule, the next run drafted "no-send, keep waiting":

```
TODO: paste the 18:21 monitor-log.md entry (FEEDBACK-IN)
TODO: paste the 18:23 monitor-log.md entry (no-send, 8 of 21 days)
```

I posted nothing.

### 7. Where it stands

As of 2026-11-15 the PR has been quiet for 13 days. GitHub reports it as mergeable and
blocked only on review. Under the playbook, the first nudge is due on 2026-11-23.

## Limitations and what's next

- **The design question is still open.** In my claim comment I asked whether the option
  should turn off only the suggestions and keep web search. Nobody answered, so the PR
  builds one reading of the issue, the one its own wording supports. **Next:** ask it as
  a yes/no question that's easy to answer in one word: "Keep Search on Google when
  suggestions are off: yes/no?"
- **"The request is skipped" comes from reading the code, not from testing.** I didn't
  capture network traffic, and I tested only on an emulator with Google as the provider.
  **Next:** test on a physical phone, and capture traffic (for example with a proxy) with
  the switch off to confirm no suggestion request goes out.
- **The monitor only recognizes bots through a rule I wrote by hand.** A new bot, or
  CodeRabbit with different wording, could trigger the same false `FEEDBACK-IN`.
  **Next:** have the monitor check the commenter's account type (GitHub's API marks bot
  accounts with `"type": "Bot"`) instead of matching on what the comment says.
