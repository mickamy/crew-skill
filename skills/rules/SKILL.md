---
name: rules
description: Build or refresh the crew rules for this repo from the user's memory and instructions, so the dispatcher and workers, who see only the memory's index, follow them.
disable-model-invocation: true
---

# crew: build the rules

The dispatcher and the workers are subagents. They load the repo's agent instructions (CLAUDE.md and the like) but, of the user's memory, only the one-line index (`MEMORY.md`), so a preference that lives only in memory reaches the agents writing the code without its why or how to apply it. `rules.md` closes that gap: the dispatcher attaches it to every ticket. This skill builds it, or brings it up to date, with the user's agreement.

## Steps

1. **Find the crew directory**, as the lead does: `.crew/` at the repo root, or `<place>/crew/` when the repo's or the user's instructions name a place for working files. If `.crew/` does not exist yet, create it with a `.gitignore` containing `*`.
2. **Gather the sources**:
   - the user's memory for this project;
   - what the user said in this conversation about how the crew should work;
   - the current `rules.md`, if any. Keep the entries the user added directly (from corrections during a run) unless the user removes them.
3. **Select** what tells an agent how to work in this repo, judged by content rather than by how the memory is labeled:
   - code conventions, libraries to use or avoid, generators to run;
   - git and pull requests: branches, commits, pull request text, which base for what;
   - tests and CI: what to run locally, what CI is used for, what not to wait for;
   - shared resources: what only one agent may use at a time, quotas, local servers;
   - responding to reviews.

   Leave out progress notes, reminders and schedules, records about people, pointers to where credentials live, and anything the repo's agent instructions already say (the agents load those anyway). Never copy a secret.
4. **Condense** each rule to one or two lines: what to do, then a short why, then the source in brackets. Group them under the headings below.
5. **Show the change before writing**: the rules added, changed, and removed against the current file, and a count of what you left out by kind, so the user can catch something missed. Write the file once the user agrees, with the build time from `date` at the top.
6. **If a crew is running in this session**, tell the dispatcher that `rules.md` changed. Tickets that are already running get the change only if the user wants it passed on.

## rules.md

```markdown
# crew rules
Built: <YYYY-MM-DD HH:MM> from memory and the user's instructions

## Code
- Use X for Y; Z broke on … [feedback-x]

## Git and pull requests
## Tests and CI
## Shared resources
## Reviews
## Other
```

Keep the whole file short enough that every worker can read it at the start of every ticket; a rule nobody reads protects nothing. If it grows past a page or two, propose moving the rules that every agent in the repo should follow into the repo's agent instructions.

## In Claude Code

- Your auto memory directory is named in your system prompt; its `MEMORY.md` indexes the files.
- When memory holds many files, hand the reading to a general-purpose subagent: give it the memory directory, the selection rules above, and the current `rules.md`, and have it return the draft. Your own context stays free for the user.
- The lead compares the memory files' modification times with the `Built:` time to suggest a rebuild, so always write that line.
