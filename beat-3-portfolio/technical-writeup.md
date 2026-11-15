# Splitting Lawnchair’s “Web suggestions” switch (#5490), and teaching my PR monitor to ignore bots

## Overview

[Lawnchair](https://github.com/LawnchairLauncher/lawnchair) is an open source Android home-screen app, also called a launcher, with nearly 14,000 stars on GitHub. The problem I worked on was in its app drawer search. A single “Web suggestions” switch controls two separate features: the autocomplete suggestions that come from the search provider, and the “Search on Google” button. If you turn the switch off, you lose both, and there’s no way to keep web search without the suggestions. [Issue #5490](https://github.com/LawnchairLauncher/lawnchair/issues/5490) asks for exactly that option. My fix adds a separate “Search suggestions” switch that is on by default, so nothing changes for anyone who leaves it alone. The whole change is 16 added lines across 4 files. For my tooling, I built a PR monitor that tracks the pull request after it opens. I wrote its playbook, and it has two rules that matter here: wait 21 quiet days before nudging the maintainers, and don’t count bot comments as feedback. Right now [PR #7477](https://github.com/LawnchairLauncher/lawnchair/pull/7477) is open and waiting for its first review from a maintainer.

## What happened

### Finding the issue

To find an issue, I built a scout that searches GitHub and grades what it finds against a rubric. Its first scan didn’t go well. It graded 5 issues and accepted none of them, because every one already had someone’s pull request linked to it. I had been filtering with `--no-assignee`, but that does nothing in repos that never assign issues to anyone. So I added `-linked:pr` to every search query, which skips issues that already have a PR. I also changed how I used the scout. Instead of relying on its search, I hand-picked issues and used it only as a grader. I screened 18 issues with quick `gh` checks, graded the 4 that were left, and #5490 ranked first on fit. The scout rated it “moderate” difficulty, because the issue says where the setting is in the app but not where it is in the code.

### Reproducing the bug

Before changing anything, I wanted to see the problem for myself. I built commit `ff03721` on the `16-dev` branch, the branch Lawnchair takes pull requests on, and pinned that commit so my “before” and “after” would compare the same code. I ran the build on an Android 16 emulator using a Pixel 7 profile, left every setting at its default, and searched `weather` in the app drawer. With “Web suggestions” on, I got three suggestions from Google and a “Search on Google” button under them. Then I turned the switch off and ran the same search. The suggestions and the “Search on Google” button both disappeared at the same time. I also tried lowering “Maximum number of suggestions” to hide them, but the slider stops at 2, so there was no setting that kept web search without the suggestions.

The code explained what I saw on the screen. One setting, `pref_searchResultStartPageSuggestion`, is read in both places:

```
PreferenceManager.kt:162:    val searchResultStartPageSuggestion = BoolPref("pref_searchResultStartPageSuggestion", true, recreate)
WebSuggestionProvider.kt:28:        if (query.isBlank() || !prefs.searchResultStartPageSuggestion.get()) {
LawnchairLocalSearchAlgorithm.kt:151:        if (prefs.searchResultStartPageSuggestion.get()) {
```

Line 162 defines the setting, and it’s on by default. Line 28 decides whether to fetch suggestions at all, and line 151 decides whether to add the “Search on Google” button. Both check the same value, so one switch could never turn off only one of them. That told me the fix needed a second, separate setting.

### Planning

Lawnchair has two places where settings can live: the older `PreferenceManager` and the newer `PreferenceManager2`. The project’s preferences docs call `PreferenceManager2` “the required location for all new settings”, so that’s where I put my new one. Before writing any code, I ran my plan through plan-check, a tool I built that compares each claim in a plan against the code and my evidence. It caught a claim I couldn’t back up: I’d written that turning the new option off “stops the network call”. That’s what the code suggests, but nothing in my testing would actually show it. So I rewrote the claim to say it comes from reading the code, and listed it as something I hadn’t tested.

### Building the fix

The core of the fix is in commit `b6ae3ea`, in `WebSuggestionProvider.kt`:

```diff
             return flow { emit(emptyList()) }
         }
 
+        if (!prefs2.showWebSearchSuggestions.firstCached()) {
+            return flow { emit(emptyList()) }
+        }
+
         val provider = prefs2.webSuggestionProvider.firstCached()
```

The four new lines check the new setting before the provider is asked for any suggestions. If the switch is off, the function returns an empty list right away, the same way the existing check just above it does. I gave the check its own `if` instead of adding it to the existing condition, so a reviewer can see the new check on its own lines.

### Testing

I wanted a unit test for this change, but Lawnchair doesn’t have a unit-test setup that covers this code. Its `tests/` folder is built for Android’s own build system, not Gradle. So I tested it the same way I first saw the bug: I re-ran the reproduction on both builds. I used the same emulator, the same search (`weather` in the app drawer), and Google as the provider. Before testing the new build, I cleared the app’s data so every setting started at its default, just like the original run.

| Setting | Before (`ff03721`) | After (`e5d0aa0`) |
|---|---|---|
| Defaults | 3 suggestions + “Search on Google” | the same |
| Web suggestions on, new Search suggestions off | not possible | “Search on Google” only, no suggestion rows |
| Web suggestions off | nothing from the web | nothing from the web (unchanged) |

The middle row is the one that proves the fix. That combination couldn’t exist before, and now it shows exactly what the issue asked for: web search without the suggestions. The other two rows check that I didn’t break anything. With the defaults, the app looks the same as before, so existing users won’t notice a change. With “Web suggestions” off, everything from the web is still hidden.

### Opening the PR

Before opening the pull request, I ran the same checks Lawnchair’s CI runs on every PR, so I wouldn’t hand the maintainers a broken build. All three passed on my branch:

- `spotlessCheck`, which checks code formatting, passed in 8 seconds.
- The GitHub debug build passed in 6 minutes 57 seconds.
- The Play debug build passed in 7 minutes 37 seconds.

Those checks run inside my ship-gate, a tool I set up that has to pass before a PR can open. Along with Lawnchair’s checks, it runs 8 of my own, such as whether the description matches the diff and whether I re-ran the reproduction before and after. Everything passed. Right after that, the script crashed while printing its summary, because the Windows console can’t print the → character. I didn’t want to trust a pass from a script that had just crashed, so I checked the gate’s log first. The pass had been recorded before the crash, and I opened PR #7477 on 2026-11-02.

### The monitor’s mistake, and the rule that fixed it

After the PR opened, my PR monitor took over. It checks the pull request, sorts it into a state, and drafts my next move, which can be “don’t send anything”. I didn’t want to guess how long to wait before nudging the maintainers, so I measured five recent Lawnchair PRs from outside contributors (#7499, #7500, #7482, #7457 and #7412). They waited 0.8, 1.3, 3.0, 12.0 and 19.6 days for a first response. I set my threshold at 21 quiet days, just past the slowest one, so I’d only nudge once the wait was longer than anything normal for this project.

On day 8, the monitor got it wrong. It labeled my PR `feedback-in`, as if a reviewer had responded. The only comment was a summary from CodeRabbit, a code-review bot, saying it had “No actionable comments”. A human would have ignored it, but the monitor counted it as feedback. So I added this rule to the playbook:

> Bot comments (CodeRabbit summaries, or a "No actionable comments" review) get no reply. Only an actionable bot finding I actually fix gets a one-line "Fixed in <commit>".

With that rule in place, the next run still labeled the PR `feedback-in`, but it recognized a bot-only thread and drafted “no-send, keep waiting” (8 of 21 days), and I posted nothing:

```
## 2026-11-10 18:21 [state] https://github.com/LawnchairLauncher/lawnchair/pull/7477

- state: feedback-in: review feedback is in (latest from the thread: comment 8 days ago)
- move: none (--state only)
- state fingerprint: e75ca0788b84
- mergeability: UNKNOWN/UNKNOWN

## 2026-11-10 18:23 [run] https://github.com/LawnchairLauncher/lawnchair/pull/7477

- state: feedback-in: review feedback is in (latest from the thread: comment 8 days ago)
- move: no-send — wait note (bot-only thread, nudge threshold unmet) (via the review response form)
- state fingerprint: ef7cd273c40f
- mergeability: MERGEABLE/BLOCKED
```

### Where it stands today

As of 2026-11-15, PR #7477 is still open. It has been quiet for 13 days, and no maintainer has reviewed it yet. GitHub reports that the PR can be merged and is only waiting on a review, so there are no conflicts or failing checks holding it back. Thirteen days is still within the range I measured for Lawnchair, where outside PRs waited up to 19.6 days for a first response. Following my playbook, I’m not nudging anyone yet. If the PR is still quiet on 2026-11-23, after 21 days, the monitor will draft a short, polite nudge.

## Limitations and what's next

**The design question is still open.** When I claimed the issue, I asked the maintainers whether the new option should turn off only the suggestions and keep web search. I haven’t had an answer yet, so the PR builds the version the issue’s own wording describes. If they meant something different, the code may need to change. Next: ask the question again in a form that’s easy to answer in one word: “Keep Search on Google when suggestions are off: yes or no?”

**Part of my claim comes from reading the code, not from testing.** The new check runs before the suggestion request, so the request should be skipped when the switch is off. But I didn’t capture network traffic to prove it. I also tested only on an emulator, with Google as the provider. Next: test on a physical phone, and capture the app’s network traffic with the switch off to confirm no suggestion request goes out.

**My monitor needed a hand-written rule to recognize a bot.** The rule names CodeRabbit and its “No actionable comments” wording, so a different bot, or new wording, could trigger the same false `feedback-in`. Next: have the monitor check the commenter’s account type instead of the comment’s text. GitHub’s API marks bot accounts as `"type": "Bot"`, which would work for any bot.
