# rAthena Configuration Reference

## Key Principle

**Never modify base config files directly.** Use `conf/import/` overrides:

```
conf/import/map_conf.txt       → overrides map_athena.conf
conf/import/char_conf.txt      → overrides char_athena.conf
conf/import/login_conf.txt     → overrides login_athena.conf
conf/import/inter_conf.txt     → overrides inter_athena.conf
conf/import/battle_conf.txt    → overrides all battle/*.conf
```

---

## inter_athena.conf (most important)

Database credentials and inter-server communication:

```
// MySQL connection
login_server_ip: 127.0.0.1
login_server_port: 3306
login_server_id: ragnarok
login_server_pw: ragnarok
login_server_db: ragnarok

char_server_ip: 127.0.0.1
char_server_port: 3306
char_server_id: ragnarok
char_server_pw: ragnarok
char_server_db: ragnarok

map_server_ip: 127.0.0.1
map_server_port: 3306
map_server_id: ragnarok
map_server_pw: ragnarok
map_server_db: ragnarok

// SQL log database (optional, separate)
log_db_ip: 127.0.0.1
log_db_id: ragnarok
log_db_pw: ragnarok
log_db_db: logs

// Use SQL item/mob databases instead of YAML
use_sql_db: no
```

---

## map_athena.conf

```
// Bind address for map server
bind_ip: 127.0.0.1

// Character server connection
char_ip: 127.0.0.1
char_port: 6121

// Public IP (what clients connect to)
map_ip: 127.0.0.1
map_port: 5121

// NPC script include file
npc: npc/scripts_main.conf

// Max connections
max_connect_user: -1  // -1 = unlimited

// Database mode: pre-renewal or renewal
// Controlled at compile time via PACKETVER
```

---

## char_athena.conf

```
// Login server connection
login_ip: 127.0.0.1
login_port: 6900

// Public address for clients
char_ip: 127.0.0.1
char_port: 6121

// Server name shown in client
server_name: My Server

// Character slots
char_per_account: 9   // Max chars per account (1-9 for most clients)

// Character creation
start_zeny: 0
start_items: 1201,1,0  // item_id,amount,is_identified

// New char start point
start_point: new_1-1,53,111   // map,x,y (renewal)
// start_point: new_2-1,53,111  // pre-renewal

// Deletion delay (days)
char_del_delay: 86400   // 24 hours = 86400 seconds
```

---

## login_athena.conf

```
// Bind port
login_port: 6900

// Allow account creation via client
new_account: yes

// Minimum password length
use_MD5_passwords: no  // Legacy MD5 (not recommended)

// IP ban settings
ipban_enable: yes
dynamic_pass_failure_ban: yes
dynamic_pass_failure_ban_count: 5
dynamic_pass_failure_ban_duration: 5  // minutes
```

---

## Battle Configuration (conf/battle/)

These files control all game balance. Key files:

### conf/battle/player.conf
```
// Experience rates
base_exp_rate: 100    // 100 = 1x, 200 = 2x, etc.
job_exp_rate: 100
mvp_exp_rate: 100

// Item drop rates
item_rate_common: 100      // Common drops
item_rate_equip: 100       // Equipment drops
item_rate_card: 100        // Card drops
item_rate_mvp_common: 100  // MVP drops
item_rate_heal: 100        // Healing item drops

// Death penalties
death_penalty_type: 0  // 0=none, 1=by level, 2=by exp

// Zeny
zeny_penalty: 0       // % zeny lost on death
```

### conf/battle/skill.conf
```
// Cast time
castrate: 100         // 100 = normal, 50 = half cast time
maxheal: 9999         // Max heal per cast
skill_delay_attack_enable: yes

// Magic cast
walkdelay: 300        // Delay (ms) after hit before walking

// Skill range
area_size: 14         // Maximum skill area size
```

### conf/battle/misc.conf
```
// General rates
autosave_time: 60000     // Auto-save interval (ms)
flooritem_lifetime: 60000 // Item on floor lifetime (ms)

// Limits
max_lv: 99            // Max base level (99/150/175 common)
max_job_lv: 70        // Max job level

// Economy
shop_increase_weight: 0   // Allow overweight shopping

// PvP
pk_mode: no           // Enable PvP mode globally
```

### conf/battle/items.conf
```
// Item behavior
item_check: yes          // Validate items on pickup
item_use_interval: 100   // Minimum ms between item uses

// Equipment
undead_detect_type: 0    // Undead detection method
```

---

## NPC Script Loading (npc/scripts_main.conf)

```
// Main NPC loader - include other conf files
npc: npc/scripts_warps.conf
npc: npc/scripts_shops.conf
npc: npc/custom/my_event.txt   // Your custom files

// Or include directly:
npc: npc/custom/my_npc.txt
```

**scripts_athena.conf** is the top-level file loaded by map server, which includes scripts_main.conf.

---

## SQL Database Tables Reference

### Core Tables (sql-files/main.sql)

| Table | Contents |
|-------|----------|
| `login` | Account credentials (id, userid, user_pass, sex, email) |
| `char` | Characters (char_id, account_id, name, class, base_level…) |
| `inventory` | Player inventory items |
| `cart_inventory` | Cart inventory |
| `storage` | Account storage |
| `guild_storage` | Guild storage |
| `guild` | Guild data |
| `party` | Party data |
| `friends` | Friends list |
| `hotkey` | Skill/item hotkeys |

### Variable Tables

| Table | Contents |
|-------|----------|
| `char_reg_num` | Character permanent integer variables |
| `char_reg_str` | Character permanent string variables |
| `acc_reg_num` | Account-local integer variables (`#var`) |
| `acc_reg_str` | Account-local string variables (`#var$`) |
| `global_acc_reg_num` | Global account integer variables (`##var`) |
| `global_acc_reg_str` | Global account string variables (`##var$`) |
| `mapreg` | Global server variables (`$var`) |

### Log Tables (sql-files/logs.sql)

| Table | Contents |
|-------|----------|
| `picklog` | Item pickup/drop/use log |
| `zenylog` | Zeny gain/loss log |
| `chatlog` | Player chat log |
| `atcommandlog` | @command usage log |
| `loginlog` | Login/logout log |
| `mvplog` | MVP kill log |

---

## Useful GM Commands

```
// Account management
@kick <charname>          Kick player
@ban <minutes> <charname> Ban for N minutes
@unban <charname>         Unban

// Character manipulation
@item <id> <amount>       Give item
@zeny <amount>            Set zeny
@blvl <level>             Set base level
@jlvl <level>             Set job level
@warp <map> <x> <y>       Warp to location
@memo <0-2>               Save memo point

// Info
@where <charname>         Show location
@whosell <item_id>        Find vending NPC with item
@mobinfo <mob_id/name>    Mob database info
@iteminfo <item_id/name>  Item database info

// World
@mapedit                  Toggle map editor
@killall                  Kill all monsters on map
@cleanmap                 Remove items from map floor
@monster <mob_id> <name>  Spawn monster at cursor
@monstersmall / @monsterbig  Size variants

// Debug
@showexp                  Toggle EXP messages
@showdelay                Toggle skill delay messages
@fakename <name>          Change visible name
```
