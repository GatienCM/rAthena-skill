# rAthena Scripting Reference

## NPC Declaration Syntax

Tab-separated fields:
```
<map>,<x>,<y>,<facing>	script	<DisplayName>::<UniqueName>	<sprite_id>,{
	/* code */
}
```

**Facing values:** 0=N, 1=NW, 2=W, 3=SW, 4=S, 5=SE, 6=E, 7=NE

**Special declarations:**
```
// Warp
<map>,<x>,<y>,0	warp	<name>	<width>,<height>,<destmap>,<destx>,<desty>

// Shop (static prices)
<map>,<x>,<y>,<facing>	shop	<name>	<sprite>,<item_id>:<price>{,<item_id>:<price>}

// Function (reusable, no map position)
function	script	<FuncName>	{
    /* code */
    return;
}

// Duplicate (clone an NPC)
<map>,<x>,<y>,<facing>	duplicate(<OriginalName>)	<NewName>	<sprite>
```

---

## Dialog Commands

```c
mes "Line of text";               // Display text
mes "Hello, "+ strcharinfo(PC_NAME) +"!";  // With variable
next;                             // Wait for player click, new dialog box
close;                            // Close dialog (end script)
close2;                           // Close dialog (continue script after)

// Menu - returns selected index in @menu
menu "Option A",label_a,"Option B",label_b,"Cancel",-;
label_a:
    mes "You chose A!";
    close;

// Or use select() which returns 1-based index
set .@choice, select("Yes","No");
if (.@choice == 1) { /* yes */ }

// Input from player
input .@value;                    // Integer input
input .@value, 1, 100;            // With min/max bounds
input .@name$;                    // String input
```

---

## Control Flow

```c
// If / else
if (condition) {
    /* code */
} else if (other) {
    /* code */
} else {
    /* code */
}

// While loop
while (.@i < 10) {
    set .@i, .@i + 1;
}

// For loop
for (set .@i, 0; .@i < 10; set .@i, .@i + 1) {
    /* code */
}

// Switch
switch (.@val) {
    case 1: mes "one"; break;
    case 2: mes "two"; break;
    default: mes "other"; break;
}

// goto (avoid when possible)
goto label_name;
label_name:
```

---

## Variable Operations

```c
set .@var, 42;                    // Assign integer
set .@str$, "hello";              // Assign string
set .@arr[0], 10;                 // Array element
set .@arr[1], 20;

// Arithmetic
set .@x, .@a + .@b * 2 - .@c / 4 % 3;

// Comparison
if (.@a == .@b) { }
if (.@a != .@b && .@a > 0) { }
if (.@a || .@b) { }

// Bitwise
set .@flags, .@flags | (1 << 3);  // Set bit 3
if (.@flags & (1 << 3)) { }       // Check bit 3
```

---

## Item Commands

```c
getitem <item_id>, <amount>;               // Give item
getitem2 <item_id>, <amount>, <identify>,  // Give with full options
    <refine>, <attribute>, <card1>, <card2>, <card3>, <card4>;
delitem <item_id>, <amount>;               // Remove item
countitem(<item_id>)                       // Count in inventory (returns int)
checkweight(<item_id>, <amount>)           // Returns 1 if player can carry
getitembound <id>, <amount>, <type>;       // Give bound item
```

---

## Player Commands

```c
// Experience
getexp <base_exp>, <job_exp>;
// OR
set BaseExp, BaseExp + 1000;

// Stats
readparam(bStr)     // Read strength
readparam(bAgi)     // Read agility
readparam(bVit)     // Read vitality
readparam(bInt)     // Read intelligence
readparam(bDex)     // Read dexterity
readparam(bLuk)     // Read luck
readparam(bMaxHp)   // Read MaxHP

// Healing
heal <hp>, <sp>;
percentheal <hp_pct>, <sp_pct>;

// Currency
set Zeny, Zeny + 10000;
if (Zeny < 1000) { mes "Not enough zeny!"; close; }

// Status effects
sc_start <effect_type>, <tick_ms>, <val1>;
sc_end <effect_type>;

// Skills
skill <skill_id>, <level>;                 // Teach skill
skilleffect <skill_id>, <level>;           // Show effect only
```

---

## Movement & Map

```c
warp "<mapname>", <x>, <y>;               // Teleport player
warpchar <account_id>, "<map>", <x>, <y>; // Teleport another player
mapwarp "<srcmap>", "<dstmap>", <x>, <y>; // Warp all on srcmap

// Map info
strcharinfo(PC_MAP)   // Current map name of player
getmapxy(.@map$, .@x, .@y, UNITTYPE_PC);  // Get position
```

---

## NPC Control

```c
enablenpc "<npcname>";
disablenpc "<npcname>";
donpcevent "<npcname>::<label>";          // Trigger event on another NPC
doevent "<npcname>::<label>";             // Same NPC
initnpctimer;                             // Start NPC timer
stopnpctimer;
setnpctimer <ms>;
getnpctimer(0)                            // Get current timer value
```

---

## Monster Spawning

```c
// Spawn monsters
monster "<map>", <x>, <y>, "<name>", <mob_id>, <count>{, "<event_label>"};
// Random position in area
areamonster "<map>", <x1>, <y1>, <x2>, <y2>, "<name>", <mob_id>, <count>;
// Remove monsters
killmonster "<map>", "<event_label>";
// All on map
killmonster "<map>", "all";
```

---

## Quest System

```c
setquest <quest_id>;                      // Start quest
completequest <quest_id>;                 // Complete quest
erasequest <quest_id>;                    // Remove quest

// Check quest state (returns QUEST_INACTIVE/QUEST_ACTIVE/QUEST_COMPLETE)
checkquest(<quest_id>)
checkquest(<quest_id>, PLAYTIME)          // Check time condition
checkquest(<quest_id>, HUNTING)           // Check hunt count
```

---

## Utility Commands

```c
// Announcements
announce "Server message!", bc_all;       // To all players
announce "Map message!", bc_map;          // Current map only
announce "Blue message!", bc_blue;        // Blue color

// Random
rand(<max>)                               // 0 to max-1
rand(<min>, <max>)                        // min to max

// String functions
strtol(.@str$, <base>)                    // String to int
.@str$ = "value: " + .@int;              // Int to string (auto)

// Time
gettimetick(2)                            // Unix timestamp
gettime(GETTIME_HOUR)                     // Current hour (0-23)
gettime(GETTIME_MINUTE)                   // Current minute
gettime(GETTIME_SECOND)

// Player info
strcharinfo(PC_NAME)                      // Character name
strcharinfo(PC_TITLE)                     // Title
getcharid(CHAR_ID_ACCOUNT)               // Account ID
getcharid(CHAR_ID_CHAR)                  // Character ID
getcharid(CHAR_ID_PARTY)                 // Party ID
getcharid(CHAR_ID_GUILD)                 // Guild ID
```

---

## Event Labels (Auto-Triggers)

```c
OnInit:         // Runs when script loads (server start / @reloadscript)
OnTouch:        // Player walks into NPC area
OnTouchNPC:     // Monster walks into NPC area
OnTimer<ms>:    // After NPC timer reaches ms (e.g. OnTimer5000: = 5 seconds)
OnClock<HHMM>:  // At specific time (e.g. OnClock1200: = noon)
OnHour<HH>:     // Every hour at HH (e.g. OnHour00: = midnight)
OnMinute<MM>:   // Every hour at minute MM

// Player events (requires player context via attachrid)
OnPCLoginEvent:
OnPCLogoutEvent:
OnPCDieEvent:
OnPCKillEvent:
OnPCBaseLvUpEvent:
OnPCJobLvUpEvent:
OnNPCKillEvent:     // When NPC mob dies (if spawned with event label)
```

---

## Calling Functions

```c
// Call a function NPC
callsub <label>;          // Within same NPC
callfunc "<FuncName>";    // Across scripts
callfunc "<FuncName>", arg1, arg2, arg3;  // With arguments

// Inside function, access args:
getarg(0)   // First argument
getarg(1)   // Second argument
getarg(0, default_val)  // With default if not provided

return;           // Return void
return <value>;   // Return value
```

---

## Full Script Command Reference

The full reference with 500+ commands is at:
`doc/script_commands.txt` in the rAthena repository.

Online: https://github.com/rathena/rathena/blob/master/doc/script_commands.txt
