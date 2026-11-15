This fall I opened my first open source fix, for Lawnchair, an Android home-screen app with nearly 14,000 stars on GitHub. It lets people turn off search suggestions while keeping web search, which wasn’t possible before because one switch controlled both.

The pull request is open and waiting for review from the project’s maintainers.

The code change was small, only 16 lines. The work around it taught me the most about how I work:

✅ I test before I claim, and I say what I didn’t test. I ran the app before and after my change to prove the fix, and I wrote down what I couldn’t confirm, like how it behaves on a physical phone.

🛠️ I used small tools I set up myself to check my own work. One runs the project’s own checks and makes sure my description matches my change before I submit. Another keeps track of the pull request afterward and tells me when a follow-up is actually due, so I don’t nudge anyone too early.

❓ When the request was unclear, I asked. It was only two sentences long, so I built the version its own wording describes and kept the new option on by default, so nothing changes for anyone who doesn’t use it.

https://github.com/LawnchairLauncher/lawnchair/pull/7477

#OpenSource #SoftwareEngineering #Android #LearningInPublic
