[claude code doc](https://docs.anthropic.com/en/docs/claude-code/overview)
# Tips for getting started
1. Run `/init` to create a `CLAUDE.md` file with 
    instructions for Claude
 2. Use Claude to help with file analysis, editing, 
    bash commands and git
 3. Be as specific as you would with another engineer
     for the best results
 4. Run /terminal-setup to set up terminal 
    integration


# General workflow advice
## Pro tips for beginners
- Be specific
- Use step-by-step instructions
- Let Claude explore first

[best practices](https://www.anthropic.com/engineering/claude-code-best-practices)

Start with planning with Opus in plan mode.
Then switch to sonnet to accomplish this plan step by step.

---

## Essential commands

Here are the most important commands for daily use:

|Command|What it does|Example|
|---|---|---|
|`claude`|Start interactive mode|`claude`|
|`claude "task"`|Run a one-time task|`claude "fix the build error"`|
|`claude -p "query"`|Run one-off query, then exit|`claude -p "explain this function"`|
|`claude -c`|Continue most recent conversation|`claude -c`|
|`claude -r`|Resume a previous conversation|`claude -r`|
|`claude commit`|Create a Git commit|`claude commit`|
|`/clear`|Clear conversation history|`> /clear`|
|`/help`|Show available commands|`> /help`|
|`exit` or Ctrl+C|Exit Claude Code|`> exit`|

- To mention a file (add to context) use `@`
- Add to memory use `#`. This will add it to the most relevant file
- `/clear` clear conversation context
- `/compact` clear history and summarize the context
- `/model` Use the Opus plan mode, and then use Sonnet
- `/vim` vim mode

`esc`, refine or 2x esc to edit previous message

`/ide`: connect to specific ide, to track the selected lines etc

shift+tab: to cycle between modes like planning
up arrow to show previous messages
you can queue messages


# Commands
You can create commands by defining `~/.claude/commands` folder, here you also can use not only basic prompts, but with variables a.k.a template

# Hooks
Hooks are user defined shell commands that will be fired based on the events
you can see them by `/hook`
You can add them by defining `~/.claude/settings.json`
You can generate them with claude code itself.
```json
{
    "hooks": {
        "Notification": [
            {
                "matcher": "",
                "hooks": [
                    {
                        "type": "command",
                        "command": "afplay \"/Users/iggysleepy/Saved sounds/wait.mp3\""
                    }
                ]
            }
        ]
    }
}
```

# Sub-agents





# Example of bad vs good commands

| Poor                                             | Good                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| add tests for foo.py                             | write a new test case for foo.py, covering the edge case where the user is logged out. avoid mocks                                                                                                                                                                                                                                                                                                                                                      |
| why does ExecutionFactory have such a weird api? | look through ExecutionFactory's git history and summarize how its api came to be                                                                                                                                                                                                                                                                                                                                                                        |
| add a calendar widget<br>                        | look at how existing widgets are implemented on the home page to understand the patterns and specifically how code and interfaces are separated out. HotDogWidget.php is a good example to start with. then, follow the pattern to implement a new calendar widget that lets the user select a month and paginate forwards/backwards to pick a year. Build from scratch without libraries other than the ones already used in the rest of the codebase. |
|                                                  |                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|                                                  |                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|                                                  |                                                                                                                                                                                                                                                                                                                                                                                                                                                         |








prompt hints:
- Remember. Your primary mission is to help me build a better game, not to please me, no matter what.
- Now, I want to share with you my game that I'm working on. And your ultimate and the only goal is to help me to create a great game. **DO NOT** agree with me just because I'm saying it. Have your stands. Always be honest and straightforward if I'm going in the wrong direction or if something sucks. Don't be delusional and don't make me delusional as well, don't treat me like I'm a child user that can't handle truth. **No matter what.**
- Don't just agree with me. Please, stop it.

