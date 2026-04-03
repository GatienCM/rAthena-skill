# rAthena Instance System

Instances are private copies of maps (and their NPCs) created per player/party/guild.
Config: `db/re/instance_db.yml`

---

## instance_db.yml Structure

```yaml
Header:
  Type: INSTANCE_DB
  Version: 2

Body:
  - Id: 1
    Name: Endless Tower
    TimeLimit: 3600        # Seconds before auto-destroy (0 = infinite)
    IdleTimeOut: 300       # Seconds with no players before destroy (0 = infinite)
    Destroyable: true      # Show destroy button in UI (default: true)
    NoNpc: false           # Don't copy NPCs into instance (default: false)
    NoMapFlag: false       # Don't copy map flags (default: false)
    Enter:
      Map: end_floor01     # Entrance map
      X: 99
      Y: 95
    AdditionalMaps:        # Extra maps duplicated with this instance
      end_floor02: true
      end_floor03: true
```

---

## Instance Modes

| Constant | Description |
|----------|-------------|
| `IM_NONE` | Not attached to anyone |
| `IM_CHAR` | Per-character instance |
| `IM_PARTY` | Per-party instance (default) |
| `IM_GUILD` | Per-guild instance |
| `IM_CLAN` | Per-clan instance |

---

## Core Commands

### Create an instance
```c
.@instance_id = instance_create("<Name>"{, <mode>{, <owner_id>}});

// Return values:
//  > 0  = instance ID (success)
//   -1  = invalid type
//   -2  = owner (party/guild/char) not found
//   -3  = instance already exists for this owner
//   -4  = no free instance slots (MAX_INSTANCE reached)
```

### Enter an instance
```c
instance_enter("<Name>"{, <x>, <y>, <char_id>, <instance_id>});

// Return values:
// IE_OK        success
// IE_NOMEMBER  party/guild not found
// IE_NOINSTANCE no instance for this owner
// IE_OTHER     other error

// Use -1,-1 for x,y to use entrance coords from instance_db.yml
```

### Destroy an instance
```c
instance_destroy {<instance_id>};
// No ID = destroy instance attached to current script
// Triggers OnInstanceDestroy: in all instance NPCs
```

### Get instance info
```c
instance_id({<mode>})                    // Get instance ID by mode
instance_mapname("<map>"{, <id>})        // Get instanced map name
instance_npcname("<npc>"{, <id>})        // Get instanced NPC name
```

### Warp all players in instance
```c
instance_warpall "<map>", <x>, <y>{, <instance_id>{, <flag>}};
// flag: IWA_NONE (default) or IWA_NOTDEAD (skip dead players)
```

### Announce in instance
```c
instance_announce <instance_id>, "<text>", bc_map;
```

### Check requirements
```c
instance_check_party(<party_id>{, <min_online>, <min_lv>, <max_lv>})
instance_check_guild(<guild_id>{, <min_online>, <min_lv>, <max_lv>})
instance_check_clan(<clan_id>{, <min_online>, <min_lv>, <max_lv>})
// Returns 1 if requirements met, 0 otherwise
```

---

## Complete Instance NPC Example

A full party instance with entrance NPC, timer, and boss:

```
// ===== ENTRANCE NPC =====
prontera,155,180,4	script	Instance Entrance::InstanceEntry	4_F_KAFRA1,{
	if (getcharid(CHAR_ID_PARTY) == 0) {
		mes "You must be in a party to enter.";
		close;
	}

	.@party_id = getcharid(CHAR_ID_PARTY);

	// Check party size and level
	if (!instance_check_party(.@party_id, 2, 50, 200)) {
		mes "Your party needs at least 2 members between level 50-200.";
		close;
	}

	// Check if instance already exists
	.@inst_id = party_instance_id(.@party_id);
	// alternative: .@inst_id = instance_id(IM_PARTY);

	if (.@inst_id <= 0) {
		// Create new instance
		.@inst_id = instance_create("My Dungeon", IM_PARTY, .@party_id);

		if (.@inst_id < 0) {
			switch (.@inst_id) {
				case -2: mes "Party not found."; break;
				case -3: mes "Your party already has an instance."; break;
				case -4: mes "Server is full. Try again later."; break;
				default: mes "Unknown error: " + .@inst_id; break;
			}
			close;
		}

		mes "Instance created! You have 60 minutes.";
	} else {
		mes "Resuming your existing instance.";
	}

	next;
	mes "Entering dungeon...";
	close2;

	// Warp player in (use instance map name)
	.@map$ = instance_mapname("my_dungeon01", .@inst_id);
	warp .@map$, 50, 50;
	end;
}

// ===== INSTANCE CONTROLLER NPC (inside the dungeon) =====
// This NPC is duplicated into each instance
my_dungeon01,0,0,0	script	DungeonController::DungeonCtrl	FAKE_NPC,{
	end;

// Runs when instance is created (after duplication)
OnInstanceInit:
	// Start a 60-minute timer
	initnpctimer;
	end;

// Timer fires every second — track time
OnTimer60000000:  // 60 minutes = 60,000,000 ms
	instance_announce instance_id(), "Time is up! The dungeon collapses.", bc_map;
	instance_warpall "prontera", 155, 180;
	instance_destroy;
	end;

// Runs when instance is destroyed
OnInstanceDestroy:
	stopnpctimer;
	end;
}

// ===== BOSS NPC =====
my_dungeon01,100,100,0	script	DungeonBoss::DungeonBoss_01	FAKE_NPC,{
	end;

OnNPCKillEvent:
	// Check if the boss monster was killed (spawn with event label)
	instance_announce instance_id(), "The boss is defeated! Congratulations!", bc_map;

	// Give rewards to all party members in instance
	.@inst_id = instance_id();
	// ... reward logic here

	// Warp everyone out after delay
	sleep 5000;
	instance_warpall "prontera", 155, 180;
	instance_destroy;
	end;
}
```

### Spawn the boss with an event label
```c
// Inside an NPC or the OnInstanceInit label:
.@map$ = instance_mapname("my_dungeon01");
monster .@map$, 100, 100, "Dark Lord", 1272, 1, strnpcinfo(NPC_NAME) + "::OnNPCKillEvent";
```

---

## Common Pitfalls

1. **Always use `instance_mapname()`** to get the correct instanced map name before warping or spawning — never use the raw map name inside an instance.
2. **`instance_npcname()`** for targeting NPCs inside the instance (same reason).
3. **`OnInstanceInit:`** is triggered after all maps and NPCs are duplicated — safe to spawn monsters there.
4. **`IdleTimeOut`** destroys the instance automatically when no players are inside — make sure it's long enough for short breaks.
5. **`instance_create()` returns -3** if the party already has an instance — always check before creating.
