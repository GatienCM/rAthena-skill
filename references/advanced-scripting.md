# rAthena Advanced Scripting

## getd / setd — Dynamic Variable Names

Access variables whose names are constructed at runtime as strings.

```c
// setd: set a variable by name string
setd "<variable_name>", <value>{, <char_id>};

// getd: get a reference to a variable by name string
getd("<variable_name>")
```

### Use cases

```c
// Dynamic variable based on player class
set getd("@bonus_" + Class), 100;
mes "Your class bonus: " + getd("@bonus_" + Class);

// Loop over array-like named variables
for (set .@i, 1; .@i <= 5; set .@i, .@i + 1) {
	setd "$event_score_" + .@i, 0;  // Reset $event_score_1 through $event_score_5
}

// Build an inventory check loop
setarray .@items[0], 501, 502, 503, 504, 505;
for (set .@i, 0; .@i < 5; set .@i, .@i + 1) {
	if (countitem(.@items[.@i]) > 0) {
		mes "You have item ID " + .@items[.@i];
	}
}

// Dynamic NPC-local storage (survives reload if permanent)
setd ".player_" + getcharid(CHAR_ID_ACCOUNT) + "_score", 500;
mes "Your score: " + getd(".player_" + getcharid(CHAR_ID_ACCOUNT) + "_score");
```

---

## Arrays — Advanced Patterns

```c
// Declare and fill
setarray .@arr[0], 10, 20, 30, 40, 50;

// Size of array
mes "Size: " + getarraysize(.@arr);   // → 5

// Copy array
copyarray .@copy[0], .@arr[0], 5;

// Delete elements
deletearray .@arr[2], 1;   // Remove index 2, shift left

// Push to end (manual)
.@arr[getarraysize(.@arr)] = 99;

// Check if value exists in array
.@found = 0;
for (set .@i, 0; .@i < getarraysize(.@arr); set .@i, .@i + 1) {
	if (.@arr[.@i] == 30) { set .@found, 1; break; }
}

// String arrays
setarray .@names$[0], "Alice", "Bob", "Charlie";
mes .@names$[1];  // → "Bob"
```

---

## bonus2 / bonus3 / bonus4 / bonus5

Used in item scripts for conditional or multi-parameter bonuses:

```c
// bonus2: bonus with one extra parameter
bonus2 bSubEle, Ele_Fire, 10;          // +10% resist fire
bonus2 bSubRace, RC_Undead, 15;        // +15% resist undead
bonus2 bSubSize, Size_Large, 5;        // +5% resist large monsters
bonus2 bAddEle, Ele_Holy, 20;          // +20% dmg vs holy element
bonus2 bAddRace, RC_DemiHuman, 10;     // +10% dmg vs demi-human
bonus2 bAddSize, Size_Medium, 8;       // +8% dmg vs medium
bonus2 bSkillAtk, "SM_BASH", 20;       // +20% Bash damage
bonus2 bSkillHeal, "AL_HEAL", 10;      // +10% Heal effectiveness
bonus2 bCastRate, "MG_FIREBALL", -20;  // -20% Fireball cast time
bonus2 bHpDrainRate, 100, 5;          // 10% chance drain 5 HP on hit
bonus2 bSpDrainRate, 100, 3;          // 10% chance drain 3 SP on hit
bonus2 bSplashRange, BF_WEAPON, 1;    // +1 splash range (weapon)
bonus2 bExpAddRace, RC_Brute, 25;     // +25% EXP from brutes

// bonus3: two extra parameters
bonus3 bAddEleClass, Ele_Fire, RC_Undead, 15;  // +15% fire vs undead
bonus3 bAutoSpell, "MG_FIREBALL", 5, 10;       // 1% chance cast Fireball Lv5 on attack
bonus3 bAutoSpellWhenHit, "AL_HEAL", 1, 100;   // 10% chance cast Heal Lv1 when hit
bonus3 bAddMonsterDropItem, PORING_CARD, 1, 100; // 0.01% Poring Card extra drop

// bonus4: three extra parameters
bonus4 bAutoSpell, "MG_FIREBALL", 5, 10, 1;   // cast on self (1) or enemy (0)

// bonus5: four extra parameters (rare, advanced)
bonus5 bAutoSpell, "MG_FIREBALL", 5, 10, BF_WEAPON, 0;
```

---

## autobonus — Proc on Attack/Hit

Triggers a bonus script for a short duration when attacking or being attacked.

```c
// autobonus "<script>", <rate>, <duration>{, <flag>{, "<trigger_script>"}};
// autobonus2 = on being hit (instead of attacking)
// autobonus3 "<script>", <rate>, <duration>, <skill_id>{, "<trigger_script>"};

// Rate: 1000 = 100% chance, 100 = 10%, 10 = 1%
// Duration: milliseconds the bonus lasts after proc

// Example: 10% chance on hit → +50 ATK for 3 seconds + fire effect
autobonus "{ bonus bAtk, 50; }", 100, 3000, BF_WEAPON,
    "{ specialeffect2 EF_FIRESPLASHHIT; }";

// Example: 5% chance on any attack → +10 all stats for 5 seconds
autobonus "{ bonus bAllStats, 10; }", 50, 5000;

// Example: 20% chance when HIT → reflect 10% damage
autobonus2 "{ bonus bShortWeaponDamageReturn, 10; }", 200, 2000, BF_SHORT;

// Flag bitmask (combine with |):
// BF_SHORT   = melee attacks only
// BF_LONG    = ranged attacks only
// (default = both)
// BF_WEAPON  = weapon attacks only
// BF_MAGIC   = magic attacks only
// BF_MISC    = misc attacks only
// (default = BF_WEAPON)
// BF_NORMAL  = normal attacks
// BF_SKILL   = skill attacks
```

---

## bonus_script — Timed Buff via Script

Apply a script-based buff to a player with full control over removal:

```c
// bonus_script "<script>", <duration>{, <flag>{, <type>{, <icon>{, <char_id>}}}};

// Simple +100 ATK for 30 seconds
bonus_script "{ bonus bAtk, 100; }", 30000;

// +STR buff, removable by Dispell, with status icon
bonus_script "{ bonus bStr, 20; }", 60000, 2, 0, SI_BLESSING;

// Flag bitmask:
// 1   = remove on death
// 2   = removable by Dispell
// 4   = removable by Clearance
// 8   = remove on logout
// 16  = removable by Banishing Buster
// 32  = removable by Refresh
// 64  = removable by Lux Anima
// 128 = remove on Madogear activation
// 256 = remove when taking damage
// 512 = permanent (not cleared by bonus_script_clear)
// 1024 = replace duplicate (extend duration)
// 2048 = stack duplicates

// Clear all bonus_scripts on player
bonus_script_clear;
bonus_script_clear 1;   // Also clear permanent ones
```

---

## Cash Shop & Points System

### NPC Declaration
```
// Static cash shop NPC
prontera,155,175,5	cashshop	Cash Shop	1_F_MARIA,512:100,513:200,514:300
// format: <item_id>:<point_cost>
```

### Point Variables
```c
#CASHPOINTS       // Account-wide cash points (account-local)
#KAFRAPOINTS      // Account-wide Kafra points (account-local)
```

### Custom Point Shop (script-based)
```c
prontera,150,170,5	script	Point Shop::PointShop	4_F_KAFRA1,{
	mes "[Point Shop]";
	mes "Your points: ^0055FF" + #CASHPOINTS + "^000000";
	next;

	.@choice = select(
		"Red Potion (100 pts)",
		"Blue Potion (200 pts)",
		"Yggdrasil Berry (500 pts)",
		"Cancel"
	);

	switch (.@choice) {
		case 1:
			if (#CASHPOINTS < 100) { mes "Not enough points."; close; }
			set #CASHPOINTS, #CASHPOINTS - 100;
			getitem 501, 1;
			mes "Purchased! Points left: " + #CASHPOINTS;
			break;
		case 2:
			if (#CASHPOINTS < 200) { mes "Not enough points."; close; }
			set #CASHPOINTS, #CASHPOINTS - 200;
			getitem 502, 1;
			mes "Purchased! Points left: " + #CASHPOINTS;
			break;
		case 3:
			if (#CASHPOINTS < 500) { mes "Not enough points."; close; }
			set #CASHPOINTS, #CASHPOINTS - 500;
			getitem 607, 1;
			mes "Purchased! Points left: " + #CASHPOINTS;
			break;
		case 4:
			close;
	}
	close;
}
```

### Add points via @command or GM NPC
```c
set #CASHPOINTS, #CASHPOINTS + 1000;   // Add 1000 cash points
set #KAFRAPOINTS, #KAFRAPOINTS + 500;  // Add 500 Kafra points
```

---

## Common Script Patterns

### Check multiple items at once
```c
// Require several items before proceeding
if (countitem(512) < 1 || countitem(513) < 5 || countitem(7227) < 10) {
	mes "You need: 1x Red Potion, 5x Blue Potion, 10x Phracon.";
	close;
}
delitem 512, 1;
delitem 513, 5;
delitem 7227, 10;
getitem 617, 1;  // Reward
```

### Dynamic menu from array
```c
setarray .@reward_items[0], 501, 502, 503, 607;
setarray .@reward_names$[0], "Red Potion", "Blue Potion", "White Potion", "Yggdrasil Berry";

.@menu$ = "";
for (set .@i, 0; .@i < getarraysize(.@reward_items); set .@i, .@i + 1) {
	set .@menu$, .@menu$ + .@reward_names$[.@i] + ":";
}
.@choice = select(.@menu$) - 1;  // 0-based
getitem .@reward_items[.@choice], 1;
```

### Cooldown system using timestamps
```c
.@cooldown = 86400;   // 24 hours in seconds

if (gettimetick(2) - @last_claim < .@cooldown) {
	.@remaining = .@cooldown - (gettimetick(2) - @last_claim);
	mes "Come back in " + (.@remaining / 3600) + "h " + ((.@remaining % 3600) / 60) + "m.";
	close;
}

set @last_claim, gettimetick(2);
getitem 607, 1;  // Daily reward
mes "Claimed! See you tomorrow.";
close;
```

### Inventory loop (check all slots)
```c
// Find all weapons in inventory
.@count = 0;
for (set .@i, 0; .@i < MAX_INVENTORY; set .@i, .@i + 1) {
	.@item_id = getinventorylist();  // Use getinventorylist() approach instead
}

// Better: use getinventorylist()
getinventorylist;
for (set .@i, 0; .@i < @inventorylist_count; set .@i, .@i + 1) {
	if (@inventorylist_type[.@i] == IT_WEAPON) {
		mes "Weapon: " + @inventorylist_id[.@i];
	}
}
```

### Safe warp with map check
```c
// Only warp if map exists and player is not in instance
if (strcharinfo(PC_MAP) == "my_map") {
	mes "You are already here!";
	close;
}
warp "my_map", 100, 100;
```

---

## Script Error Debugging

### Common errors and fixes

| Error in map-server log | Cause | Fix |
|------------------------|-------|-----|
| `player not attached` | Using player command with RID=0 | Add `attachrid()` or check `playerattached()` |
| `unknown variable type` | Wrong variable prefix | Check scope prefix (`.@`, `@`, `$`, `#`, etc.) |
| `parse error near ','` | Syntax error in script | Check brackets, semicolons, tabs in NPC header |
| `npc not found` | Wrong NPC name in `donpcevent` | Match exact `::UniqueName` including case |
| `duplicate NPC name` | Two NPCs with same unique name | Rename one |
| `stack overflow` | Infinite callsub/callfunc loop | Add exit condition |

### Debug techniques
```c
// Print variable value to player
mes "Debug: .@val = " + .@val;

// Print to map-server console
debugmes "Checkpoint reached, val=" + .@val;

// Check if player is attached
if (!playerattached()) {
	debugmes "No player attached — skipping";
	end;
}

// Dump all inventory
getinventorylist;
debugmes "Inventory count: " + @inventorylist_count;
```
