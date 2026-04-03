---
name: rathena
description: >
  Expert assistant for rAthena Ragnarok Online server emulator development.
  Use this skill whenever the user mentions rAthena, eAthena, Ragnarok Online server,
  NPC scripts, item_db, mob_db, skill_db, atcommands, map server, char server, login server,
  or any .txt script files for an RO server. Covers scripting (NPC, quests, shops, warps),
  YAML database editing (items, mobs, skills), C++ source modifications, configuration files,
  build system (CMake), SQL schema, packet handling, and server administration.
  Always trigger this skill when the user works on an RO private server or asks about
  rAthena scripting, custom content, or server-side development.
---

# rAthena Development Guide

rAthena is an open-source Ragnarok Online server emulator written in C++14.
It runs as **4 independent processes** communicating via TCP + MySQL:

| Server | Port | Role |
|--------|------|------|
| Login Server | 6900 | Auth, IP banning, session tokens |
| Character Server | 6121 | Character CRUD, inter-server relay |
| Map Server | 5121 | World simulation, NPCs, skills, battle |
| Web Server | HTTP | Guild emblems, merchant config APIs |

**Data split:**
- **Static game data** → YAML files in `db/re/` (renewal) or `db/pre-re/` (pre-renewal)
- **Dynamic player data** → MySQL (account, char, inventory, guild, storage…)

## Quick Reference

| Topic | File |
|-------|------|
| NPC scripting (dialog, variables, events, quests) | `references/scripting.md` |
| Advanced scripting (getd/setd, autobonus, arrays, cash shop, debug) | `references/advanced-scripting.md` |
| Complete NPC examples (quest, daily, shop, event boss, refine...) | `references/examples.md` |
| YAML databases (item_db, mob_db, skill_db, mob_avail) | `references/databases.md` |
| Map flags (setmapflag, all mf_* constants) | `references/mapflags.md` |
| Instance system (instance_create, instance_enter...) | `references/instances.md` |
| WoE / GvG / Battleground | `references/woe-gvg.md` |
| C++ development & packets | `references/cpp-patterns.md` |
| Configuration files (.conf, SQL tables, GM commands) | `references/configuration.md` |

---

## Common Tasks

### Add a custom item
1. Edit `db/re/item_db_equip.yml` (equipment) or `item_db_usable.yml` / `item_db_etc.yml`
2. Reload in-game: `@reloaditemdb` (GM command), or restart map server
3. Give via script: `getitem <item_id>, <amount>;`

### Add a custom monster
1. Edit `db/re/mob_db.yml`
2. Spawn via script: `monster "<map>", <x>, <y>, "<name>", <mob_id>, <count>;`
3. Reload: `@reloadmobdb`

### Add a custom skill
1. Define in `db/re/skill_db.yml`
2. Implement C++ in `src/map/skill_factory_*.cpp` (unity build pattern)
3. Register handler in execution pipeline via `skill_castend_id()`

### Create an NPC
```
prontera,150,150,4	script	MyNPC::MyNPC_ID	4_F_KAFRA1,{
	mes "Hello, "+ strcharinfo(PC_NAME) +"!";
	next;
	mes "Welcome to my shop.";
	close;
}
```

### Create a warp
```
prontera,100,100,0	warp	to_moc	1,1,morocc,156,93
```

### Reload without restart
| Command | Effect |
|---------|--------|
| `@reloaditemdb` | Reload item database |
| `@reloadmobdb` | Reload monster database |
| `@reloadskilldb` | Reload skill database |
| `@reloadscript` | Reload all NPC scripts |
| `@reloadbattleconf` | Reload battle config |

---

## Script Variable Scoping

| Prefix | Scope | Persistence | Example |
|--------|-------|-------------|---------|
| (none) | Character | Permanent (SQL) | `questvar` |
| `@` | Character | Temporary (session) | `@temp` |
| `#` | Account (local server) | Permanent | `#CashPoints` |
| `##` | Account (global) | Permanent | `##GlobalVar` |
| `.@` | NPC scope-local | Temporary | `.@i` |
| `.` | NPC | Until server restart | `.npcstate` |
| `$` | Global | Permanent (mapreg) | `$EventStatus` |
| `$@` | Global | Temporary | `$@temparray` |

- Integer variables: no suffix → `@gold`
- String variables: `$` suffix → `@name$`
- Arrays: indexed 0..2,147,483,647

---

## Key Read-Only Script Variables

```
Zeny, BaseLevel, JobLevel, Class, Sex
Hp, MaxHp, Sp, MaxSp
Weight, MaxWeight
BaseExp, JobExp
strcharinfo(PC_NAME), strcharinfo(PC_MAP)
```

---

## Build System (CMake)

```bash
# Configure
cmake -B build -DPACKETVER=20211103

# Key flags
-DPACKETVER=<date>         # Client packet version
-DENABLE_WEB_SERVER=ON     # Include web server
-DENABLE_EXTRA_DEBUG_CODE=ON

# Build
cmake --build build -j$(nproc)

# Tools
make import   # Create conf/import/ directories
```

**Legacy:**
```bash
./configure && make server
```

---

## Directory Structure

```
rathena/
├── conf/               # All configuration files
│   ├── import/         # Local overrides (don't touch base configs)
│   ├── battle/         # Balance tuning
│   ├── map_athena.conf
│   ├── char_athena.conf
│   ├── login_athena.conf
│   └── inter_athena.conf  # MySQL credentials + inter-server
├── db/
│   ├── re/             # Renewal databases (YAML)
│   └── pre-re/         # Pre-renewal databases (YAML)
├── npc/                # NPC script files (.txt)
├── src/
│   ├── map/            # Map server (most complex)
│   ├── char/           # Character server
│   ├── login/          # Login server
│   ├── web/            # Web server
│   └── common/         # Shared utilities
├── sql-files/          # Database schema SQL files
└── doc/
    └── script_commands.txt  # Full script command reference (~500k lines)
```

---

## Ports & Identifiers

- `RID` = current player's Account ID in script context (0 = no player)
- `AID` = Account ID in login DB
- `PACKETVER` = client protocol version (build-time flag)
- `attachrid(<account_id>)` — attach player context for server-side events
- `detachrid` — release player context
