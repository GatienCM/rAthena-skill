# rAthena Skill for Claude Code

A Claude Code skill that gives Claude expert knowledge of the [rAthena](https://github.com/rathena/rathena) Ragnarok Online server emulator.

Built from the [rAthena Deep Wiki](https://deepwiki.com/rathena/rathena).

---

## What it covers

- **Architecture** — 4-server model (login, char, map, web), ports, data split between YAML and MySQL
- **NPC Scripting** — Full language reference: dialog, menus, variables, control flow, events, quests, item/player/monster commands
- **YAML Databases** — `item_db`, `mob_db`, `skill_db`, `mob_avail` (player sprite on monster) with all fields and examples
- **C++ Development** — Adding custom script commands (`buildin_*`), `@atcommands`, skill implementations, packet handling, timers, SQL queries
- **Configuration** — All `.conf` files (`inter`, `map`, `char`, `login`, `battle/*`), SQL table reference, GM commands

---

## Installation

### Global (available in all your projects)

```bash
git clone https://github.com/GatienCM/rAthena-skill.git
cp -r rAthena-skill/. ~/.claude/skills/rathena/
```

### Project-level (only for a specific project)

```bash
mkdir -p .claude/skills/rathena
cp -r rAthena-skill/. .claude/skills/rathena/
```

---

## Usage

Once installed, the skill triggers automatically when you mention:
- rAthena, eAthena, Ragnarok Online server
- NPC scripts, item_db, mob_db, skill_db
- map server, char server, login server
- atcommands, script commands, warps, shops

No manual activation needed — just ask your question and Claude will use the skill context.

---

## File structure

```
rathena/
├── SKILL.md                  # Main skill file (architecture, quick ref, common tasks)
└── references/
    ├── scripting.md           # Full NPC scripting language reference
    ├── databases.md           # YAML databases: item_db, mob_db, skill_db, mob_avail
    ├── cpp-patterns.md        # C++ development: commands, packets, skills, timers
    └── configuration.md       # Config files, SQL tables, GM commands
```

---

## Compatibility

- Claude Code (global `~/.claude/skills/` or project `.claude/skills/`)
- Cursor (`~/.cursor/skills/`)
- Codex (`~/.codex/skills/`)
