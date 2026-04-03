# rAthena Skill for Claude Code

A Claude Code skill that gives Claude expert knowledge of the [rAthena](https://github.com/rathena/rathena) Ragnarok Online server emulator.

Built from the [rAthena Deep Wiki](https://deepwiki.com/rathena/rathena) and the [official rAthena repository](https://github.com/rathena/rathena).

---

## What it covers

| Reference | Contents |
|-----------|----------|
| **Scripting** | Dialog, menus, variables, control flow, events, quests, item/player/monster commands |
| **Advanced Scripting** | `getd`/`setd`, arrays, `autobonus`, `bonus2/3`, cash shop, cooldowns, debug techniques |
| **NPC Examples** | 8 complete ready-to-use scripts: quest, daily reward, point shop, event boss, warp portal, refine NPC, login events |
| **YAML Databases** | `item_db`, `mob_db`, `skill_db`, `mob_avail` (player sprite on monster), `quest_db` |
| **Map Flags** | All `mf_*` constants with descriptions and practical examples |
| **Instance System** | `instance_create`, `instance_enter`, `instance_destroy`, full dungeon NPC template |
| **WoE / GvG** | WoE FE/SE/TE scheduling, GvG map control, Battleground, guild utilities |
| **C++ Development** | Custom script commands (`buildin_*`), `@atcommands`, skill implementations, packet handling, timers |
| **Configuration** | All `.conf` files (`inter`, `map`, `char`, `login`, `battle/*`), SQL table reference, GM commands |

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
- NPC scripts, `item_db`, `mob_db`, `skill_db`
- map server, char server, login server
- `@atcommands`, script commands, warps, shops, WoE, instances

No manual activation needed — just ask your question and Claude will use the skill context.

---

## File Structure

```
rathena/
├── SKILL.md                        # Main skill (architecture, quick ref, common tasks)
└── references/
    ├── scripting.md                # Full NPC scripting language reference
    ├── advanced-scripting.md       # getd/setd, autobonus, bonus2/3, cash shop, debug
    ├── examples.md                 # 8 complete ready-to-use NPC scripts
    ├── databases.md                # item_db, mob_db, skill_db, mob_avail, quest_db
    ├── mapflags.md                 # All mf_* map flags with examples
    ├── instances.md                # Instance system with full dungeon template
    ├── woe-gvg.md                  # WoE, GvG, Battleground
    ├── cpp-patterns.md             # C++ development: commands, packets, skills, timers
    └── configuration.md            # Config files, SQL tables, GM commands
```

---

## Compatibility

- Claude Code (global `~/.claude/skills/` or project `.claude/skills/`)
- Cursor (`~/.cursor/skills/`)
- Codex (`~/.codex/skills/`)
