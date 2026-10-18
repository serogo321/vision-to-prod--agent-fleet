# Install

Paste everything below the line into Claude Code, as-is.

---

Install the **vision-to-prod--agent-fleet** subagents from https://github.com/serogo321/vision-to-prod--agent-fleet. Follow these steps in order:

1. **Ask me where to install them** and wait for my answer:
   - **for me, in every project:** `~/.claude/agents/`
   - **for this project only:** `.claude/agents/` in the current repository.

   A project-level file overrides a user-level file with the same name.
2. **Get the files.** Clone the repository into a temporary directory with
   `git clone --depth 1 https://github.com/serogo321/vision-to-prod--agent-fleet <temp dir>`. If I give you a local path to a copy
   instead, use that. You only need the `.md` files in its `agents/` folder.
3. **Check for name clashes before copying anything.** For each file in `agents/`:
   - **If the target folder has no file with that name,** it is a new agent: copy it.
   - **If the target folder already has a file with that name,** do NOT overwrite it yet. Show me the existing file's
     `name`, `description` and `model` lines next to the new one's, and ask me whether to overwrite it, keep mine, or
     save the new one under a different name. Ask once per clashing file, and do what I answer.
4. **Change nothing else.** Don't edit the contracts while copying, and don't touch other files in the target folder,
   my settings, or my CLAUDE.md.
5. **Confirm the agents load:**
   - Check that each installed file starts with valid YAML frontmatter that has `name`, `description`, `model`,
     `effort` and `tools`. Check that its `name` matches its file name.
   - Then list the agent types available to you and confirm each installed agent appears.
   - If they do not appear yet, tell me to restart Claude Code, or to open `/agents`. Agent definitions can be loaded
     once per session, so new files may only show up in a new session.
6. **Report back:**
   - which agents were installed, skipped or overwritten;
   - any agent whose `model:` line names a model my account cannot use. Tell me to change that line to a model I
     have. Each agent pins its model and effort in its frontmatter; dispatch it WITHOUT an inline `model` parameter,
     because an inline model overrides the pin;
   - that `project-manager` must be launched WITHOUT a `name`. With agent teams on, a named launch makes it a
     teammate, and its lanes can't hand their reports back to it;
   - the optional tools and skills the README lists as "if available". Note which of them I seem to have.
7. **Delete the temporary clone.**
