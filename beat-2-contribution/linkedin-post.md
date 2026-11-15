I just opened my first open source contribution: a fix for Lawnchair, an Android home-screen app with 13.7k stars on GitHub. It lets people turn off search suggestions while keeping web search, which used to be impossible because one switch controlled both.

The pull request is open and awaiting review from the project's maintainers.

The code change was small. The work around it taught me the most about how I work:

✅ I test before I claim, and I say what I didn't test. I checked the fix by running the app before and after the change. I also wrote down what I couldn't confirm, such as how it behaves on a physical phone.

🛠️ I build small tools to check my own work. One runs the project's own checks and reviews my description against the actual change before I submit. Another tracks the pull request afterward and tells me when a follow-up is actually due, so I don't nudge too early.

❓ When the request was unclear, I asked. The original request was two sentences long. I built the version its own wording describes and made the new option on by default, so nothing changes for anyone who doesn't use it.

https://github.com/LawnchairLauncher/lawnchair/pull/7477

#OpenSource #SoftwareEngineering #Android #LearningInPublic
