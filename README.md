# scriptcase-agent-skill

An [Agent Skill](https://agentskills.io/) that teaches AI coding agents how to help you develop
**ScriptCase Blank applications** (`onExecute`) from a local PHP draft that you paste into
ScriptCase.

The skill is generic and contains no client/project data. It works with any Agent Skills–compatible
client (opencode, Claude Code, Cursor, GitHub Copilot, Gemini CLI, and more).

## What's inside

```
skills/scriptcase/
├── SKILL.md        # Main guide
├── reference.md    # Complete copy-ready draft example
└── macros.md       # Catalog of blank-app-safe built-in macros
```

`SKILL.md` covers:

- The draft format: no leading `<?php`, use `?>` for HTML, end with an open `<?php`.
- The three variable families: `[global]` (session), `{composite}` variable, `$local`.
- SQL macros: `sc_lookup()`, `sc_select()`, `sc_exec_sql()` — result access, scope rules, connections.
- Transactions, error/alert/redirect macros, form POST add/edit, user-defined functions.
- Security, debugging, common pitfalls, and a pre-paste checklist.
- Appendix with the most useful built-in macros and runtime objects.

`macros.md` lists the blank-app-safe macros by category (SQL/data, error/log, session/global,
navigation/messages, include/library, date/number/encoding, email/API, auth/LDAP, files) and calls
out the grid/form/button macros that do **not** apply to a Blank application.

## Install

### opencode

Copy the skill folder into one of the discovery locations:

```
# Project (shared with the repo)
.opencode/skills/scriptcase/SKILL.md

# Global
~/.config/opencode/skills/scriptcase/SKILL.md
```

Or point opencode at this repo's `skills/` folder in `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": { "paths": ["/absolute/path/to/scriptcase-agent-skill/skills"] }
}
```

Then restart opencode.

### Other Agent Skills clients

Copy `skills/scriptcase/` into the client's skills directory
(e.g. `.claude/skills/scriptcase/`, `.agents/skills/scriptcase/`).

## License

MIT — see [LICENSE](LICENSE).
