# rAthena C++ Development Reference

## Source Structure

```
src/
├── map/            # Map server — world simulation
│   ├── clif.cpp/h      # Client ↔ server packet routing
│   ├── skill.cpp/h     # Skill execution pipeline
│   ├── battle.cpp/h    # Damage calculation
│   ├── status.cpp/h    # Buffs/debuffs, stats
│   ├── script.cpp/h    # NPC script VM
│   ├── mob.cpp/h       # Monster AI state machine
│   ├── itemdb.cpp/h    # Item database
│   ├── pc.cpp/h        # Player character logic
│   ├── map.cpp/h       # Map data and cell management
│   ├── guild.cpp/h     # Guild system
│   ├── party.cpp/h     # Party system
│   ├── storage.cpp/h   # Storage system
│   ├── atcommand.cpp/h # @commands implementation
│   └── skill_factory_*.cpp  # Skill implementations (unity build)
├── char/           # Character server
│   ├── char.cpp/h      # Main char server logic
│   └── char_mapif.cpp  # Map ↔ char inter-server
├── login/          # Login server
│   └── login.cpp/h
├── web/            # HTTP web server
└── common/         # Shared utilities
    ├── database.hpp    # TypesafeYamlDatabase base
    ├── timer.cpp/h     # Timer system
    ├── socket.cpp/h    # TCP socket layer
    ├── sql.cpp/h       # MySQL wrapper
    └── malloc.cpp/h    # Memory manager
```

---

## Key Architectural Patterns

### TypesafeYamlDatabase (game data loading)

All YAML databases inherit from this template:

```cpp
// base class in src/common/database.hpp
template<typename K, typename V>
class TypesafeYamlDatabase {
public:
    virtual bool parseBodyNode(const ryml::NodeRef& node) = 0;
    bool load();
    bool reload();
    std::shared_ptr<V> find(K id);
};

// Example: ItemDatabase in src/map/itemdb.hpp
class ItemDatabase : public TypesafeYamlDatabase<uint32, item_data> {
    bool parseBodyNode(const ryml::NodeRef& node) override;
};
extern ItemDatabase itemdb;

// Usage in scripts/code:
auto item = itemdb.find(item_id);
auto item = itemdb.search_aegisname("Poring_Card");
```

### Adding a Custom Script Command (buildin_*)

1. Declare in `src/map/script.cpp`:
```cpp
// In the buildin_func_table[] array:
BUILDIN_FUNC(mycommand),
```

2. Implement the function:
```cpp
BUILDIN_FUNC(mycommand) {
    // Get arguments from script stack
    int32 arg1 = script_getnum(st, 2);      // 1st arg (index starts at 2)
    const char* arg2 = script_getstr(st, 3); // 2nd arg as string

    // Optional arg with default
    int32 opt = script_hasdata(st, 4) ? script_getnum(st, 4) : 0;

    // Access current player
    map_session_data* sd = script_rid2sd(st);
    if (!sd) return SCRIPT_CMD_FAILURE;

    // Do your logic here
    sd->status.zeny += arg1;

    // Return a value (optional)
    script_pushint(st, 1);  // push int result
    // script_pushstr(st, "result"); // push string result

    return SCRIPT_CMD_SUCCESS;
}
```

3. Register in the table:
```cpp
// In script_buildin[] in script.cpp:
{ "mycommand", buildin_mycommand, "i??" },
// Format string: i=int, s=string, r=reference, ?=optional, *=variadic
```

### Adding an @command (atcommand)

In `src/map/atcommand.cpp`:

```cpp
ACMD_FUNC(mycommand) {
    // message[] contains the argument string after @mycommand
    if (!*message) {
        clif_displaymessage(fd, "Usage: @mycommand <arg>");
        return false;
    }

    int32 value = atoi(message);
    // Access the player: sd is already provided
    sd->status.base_level = value;
    status_calc_pc(sd, SCO_FORCE);

    clif_displaymessage(fd, "Done!");
    return true;
}

// Register in atcommand_base_list[]:
{ "mycommand", 40, 40, ACMD(mycommand) },
// { name, player_level, gm_override_level, function }
```

### Skill Implementation (unity build pattern)

Skills are implemented in factory files via `#include`:

```cpp
// src/map/skill_factory_knight.cpp
namespace rathena {

// Individual skill file included:
// #include "skills/SM_BASH.cpp"

static std::unique_ptr<const SkillImpl> SM_BASH_create(uint16 skill_id) {
    return std::make_unique<SmBash>();
}

} // namespace rathena
```

Each skill class:
```cpp
class SmBash : public SkillImpl {
public:
    bool cast(map_session_data* sd, block_list* target, uint16 lv) override {
        // Validate
        if (!target) return false;

        // Calculate damage
        int32 damage = battle_calc_attack(BF_WEAPON, &sd->bl, target, skill_id, lv, 0);

        // Apply
        skill_attack(BF_WEAPON, &sd->bl, &sd->bl, target, skill_id, lv, gettick(), 0);
        return true;
    }
};
```

---

## Packet System

### Naming Conventions
- `PACKET_CZ_*` = Client → Server (CZ = Client to Zone/Map)
- `PACKET_ZC_*` = Server → Client (ZC = Zone to Client)
- `PACKET_CH_*` = Server → Char server
- `PACKET_HC_*` = Char → Login server

### Packet Structure (src/map/packets_struct.hpp)
```cpp
#pragma pack(push, 1)
struct PACKET_CZ_REQUEST_MOVE {
    int16 PacketType;   // 0x0085
    pos_dir_coord dest; // destination coordinates
};
#pragma pack(pop)
```

### Reading a Packet (in clif.cpp)
```cpp
void clif_parse_WalkToXY(int32 fd, map_session_data* sd) {
    auto* p = reinterpret_cast<PACKET_CZ_REQUEST_MOVE*>(RFIFOP(fd, 0));
    // p->dest.x, p->dest.y, p->dest.dir
    unit_walktoxy(&sd->bl, p->dest.x, p->dest.y, p->dest.dir);
}
```

### Sending a Packet
```cpp
// Send to one client
clif_displaymessage(sd->fd, "Hello!");

// Send to area (all players in range)
clif_send(packet_buf, sizeof(packet_buf), &sd->bl, AREA);

// Send targets: SELF, AREA, PARTY, GUILD, ALL_CLIENT, BL_PC, etc.
```

---

## Socket I/O Macros

```cpp
// Read from socket buffer
RFIFOP(fd, offset)          // Pointer to data at offset
RFIFOB(fd, offset)          // Read 1 byte
RFIFOW(fd, offset)          // Read 2 bytes (little-endian)
RFIFOL(fd, offset)          // Read 4 bytes
RFIFOSKIP(fd, size)         // Advance read pointer

// Write to socket buffer
WFIFOP(fd, offset)          // Pointer to write at offset
WFIFOB(fd, offset, val)     // Write 1 byte
WFIFOW(fd, offset, val)     // Write 2 bytes
WFIFOL(fd, offset, val)     // Write 4 bytes
WFIFOSET(fd, size)          // Commit write
```

---

## Status Effects (sc_start / sc_end)

```cpp
// Apply status effect
sc_start(&bl, SC_BLESSING, 100, level, duration_ms);
// sc_start(block_list*, type, rate%, val1, duration)

// Remove
status_change_end(&bl, SC_BLESSING);

// Check
status_change* sc = status_get_sc(&bl);
if (sc && sc->data[SC_BLESSING]) {
    int32 lv = sc->data[SC_BLESSING]->val1; // blessing level
}
```

---

## Timer System

```cpp
// Add a timer
t_tick tick = add_timer(gettick() + 5000, my_timer_func, id, data);

// Timer callback signature
TIMER_FUNC(my_timer_func) {
    // id = attached entity ID
    // data = custom data passed at creation
    // Returns 0 on success
    return 0;
}

// Delete timer
delete_timer(tick, my_timer_func);

// Interval timer (repeating)
t_tick tick = add_timer_interval(gettick(), my_func, id, data, 1000); // every 1s
```

---

## SQL Queries (common/sql.hpp)

```cpp
SqlConnection* sql_handle; // global connection

// Simple query
if (SQL_ERROR == Sql_Query(sql_handle, "SELECT `name` FROM `char` WHERE `char_id`=%d", char_id)) {
    Sql_ShowDebug(sql_handle);
    return;
}

// Fetch rows
if (SQL_SUCCESS == Sql_NextRow(sql_handle)) {
    char* name;
    Sql_GetData(sql_handle, 0, &name, nullptr);
    // use name
}

Sql_FreeResult(sql_handle);
```

---

## Important Constants & Enums

```cpp
// Block list types (bl.type)
BL_PC   = 1   // Player
BL_MOB  = 2   // Monster
BL_PET  = 4   // Pet
BL_HOM  = 8   // Homunculus
BL_MER  = 16  // Mercenary
BL_ITEM = 32  // Item on ground
BL_SKILL= 64  // Skill unit (e.g., traps)
BL_NPC  = 128 // NPC
BL_CHAT = 256 // Chat room
BL_ALL  = 0xFFF

// Battle flags
BF_WEAPON  = 1  // Physical weapon attack
BF_MAGIC   = 2  // Magic attack
BF_MISC    = 4  // Misc (traps, etc.)
BF_SHORT   = 16 // Melee range
BF_LONG    = 64 // Ranged

// Skill cast return
SKILL_ERROR  = 0
SKILL_CASTEND = 1

// Status change important values
SC_BLESSING, SC_AGI, SC_INCREASEAGI, SC_ENDURE,
SC_HIDING, SC_CLOAKING, SC_PROVOKE, SC_ASPERSIO,
SC_FREEZE, SC_STUN, SC_SLEEP, SC_SILENCE, SC_BLIND,
SC_POISON, SC_CURSE, SC_CONFUSION
```

---

## Debugging Tips

```bash
# Enable debug build
cmake -B build -DCMAKE_BUILD_TYPE=Debug -DENABLE_EXTRA_DEBUG_CODE=ON

# AddressSanitizer for memory errors
cmake -B build -DCMAKE_CXX_FLAGS="-fsanitize=address -g"

# Log levels in conf/map_athena.conf
log_filter: 1  # 0=none, 1=errors, 2=warnings, 3=info, 4=debug

# GDB attach to running server
gdb -p $(pidof map-server)
```

---

## Common Mistakes

1. **Forgetting `status_calc_pc(sd, SCO_FORCE)`** after modifying player stats — changes won't apply until recalculated.
2. **Not calling `clif_updatestatus()`** after Zeny/HP/SP changes — client won't reflect the update.
3. **RID = 0 in server events** — always `attachrid()` before using player commands in `OnInit` or timer events.
4. **YAML indentation** — rAthena YAML is strict; use spaces not tabs.
5. **`@reloadscript` clears NPC variables** — `.npc_var` values are lost on reload; use permanent variables (`npc_var`) if persistence is needed across reloads.
