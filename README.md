# crew

A Claude Code plugin that runs a crew of agents on your repository. You talk to one agent, the lead. A dispatcher hands tickets to workers, and each worker turns one ticket into a pull request in its own git worktree. Nothing merges without your approval.

```
you ⇄ lead            the only agent you talk to; plans, asks, merges what you approve
        ⇅
      dispatcher       hands out tickets, takes reports, forwards what needs you
        ├ worker       one ticket → one pull request, in its own worktree
        ├ worker
        └ worker
```

## Why

- **One conversation.** Questions and review requests from every worker reach you through the lead, one decision at a time, each with the background to decide it and the lead's recommendation.
- **Your word, and only yours, merges.** Claude Code never treats a message from one agent as your consent, so crew does not try to relay approvals. The lead, the agent that hears you, is the only one that merges, closes pull requests, or cancels CI runs.
- **Your attention goes to decisions and reviews.** The dispatcher answers what is already settled, checks each pull request's scope and CI before you see it, and keeps the workers busy while you think.
- **Built on Claude Code's own plumbing.** Subagents, worktrees, and messaging come from Claude Code. crew carries the working rules on top of them.

## Requirements

- Claude Code (tested with v2.1.319)
- git, and the GitHub CLI (`gh`) signed in to the repository's host
- A GitHub repository with pull requests and, ideally, CI

## Install

```
/plugin marketplace add mickamy/crew-skill
/plugin install crew@crew-skill
```

To try a local checkout instead: `claude --plugin-dir /path/to/crew-skill`.

## Use

| Command | What happens |
| --- | --- |
| `/crew:start` | The session becomes the lead. Nothing starts until you say what to work on. If an earlier run left a board, the lead says so. |
| `/crew:start <goal>` | The lead splits the goal into tickets, each one pull request, and asks you to approve the list. |
| `/crew:rules` | Builds `rules.md` from your memory and instructions, after showing you the change. |

You can also ask the lead to look for work ("find things worth fixing in the sync code"). It sends read-only scouts over the repo and shows you a ranked list.

Answer by ID, in any order: `T3 OK`, `T3-Q1: B`, `all OK` for a list the lead just showed. If you say you are stepping away and want the crew to keep going, the lead decides small questions on its own recommendation and hands you that list when you return. Merges still wait for you.

## Approving merges

A pull request counts as approved when any one of these holds:

1. You tell the lead, naming the pull requests.
2. Your repository has both labels below, and you add `user-approved` to the pull request. This lets you approve from GitHub, phone included. It is off until you create the labels:

   ```
   gh label create needs-user-review --color fbca04 --description "Waiting for the user's review before an agent merges it"
   gh label create user-approved --color 0e8a16 --description "The user approved this for an agent to merge"
   ```

3. The pull request was opened by a different account from yours (a bot, for example), and you submit an approving GitHub review.

An approval goes stale when commits that change the content land after it, and the lead asks again.

## What crew reads and writes

- **Reads**: the repository's instructions for agents (CLAUDE.md, AGENTS.md), CONTRIBUTING, docs, and workflows. Commit style, pull request format, merge method, and tests follow your repository.
- **Writes**: `.crew/board.md` (tickets and decisions, written by the lead) and `.crew/rules.md`, with a `.gitignore` inside `.crew/` that ignores the directory itself, so no tracked file changes. If your instructions name a place for working files, crew uses `<that place>/crew/` instead.
- **On GitHub**: branches and pull requests for the tickets, and the two labels above only if you created them.

Workers are subagents, and subagents do not see your memory. Run `/crew:rules` so that the preferences you keep in memory reach them.

## Good to know

- Permission prompts from every agent appear in the lead's terminal. Allow rules for the git and `gh` commands you are comfortable with keep the crew moving.
- Three workers run at once by default; tell the lead if you want a different number.
- The crew lives in the lead's session. Closing it stops the crew; resume the session to continue.
- Tested only with Claude Code. Other agents (Codex and the like) are not supported yet.

## License

[MIT](LICENSE)
