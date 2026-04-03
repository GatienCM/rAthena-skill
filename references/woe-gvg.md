# rAthena WoE / GvG System

## War of Emperium Overview

Three WoE versions:
- **WoE FE** (First Edition) — Original castles
- **WoE SE** (Second Edition) — Renewal castles
- **WoE TE** (Tactical Emperium) — Smaller-scale battles

---

## Start / Stop Commands

```c
agitstart;    // Start WoE FE
agitend;      // End WoE FE

agitstart2;   // Start WoE SE
agitend2;     // End WoE SE

agitstart3;   // Start WoE TE
agitend3;     // End WoE TE
```

These trigger `OnAgitStart:` / `OnAgitEnd:` labels across **all NPCs** on the server.

### Check WoE status
```c
agitcheck()   // Returns 1 if WoE FE is active
agitcheck2()  // Returns 1 if WoE SE is active
agitcheck3()  // Returns 1 if WoE TE is active
```

---

## GvG Map Control

```c
gvgon "<map>";     // Enable GvG on map (FE/SE)
gvgoff "<map>";    // Disable GvG on map

gvgon3 "<map>";    // Enable GvG TE on map
gvgoff3 "<map>";   // Disable GvG TE on map
```

---

## Guild Utilities

```c
// Warp guild/non-guild players, clean mobs on map
maprespawnguildid "<map>", <guild_id>, <flag>;
// Flags (combine with |):
//  1 = warp guild members to save point
//  2 = warp non-guild members to save point
//  4 = remove non-guardian monsters
//  7 = all of the above (full takeover cleanup)

// Get alliance status between two guilds
getguildalliance(<guild_id1>, <guild_id2>)
// Returns: -2 (enemy), -1 (hostile), 0 (none), 1 (ally), 2 (same guild)

// Set emblem on a guild flag NPC (sprite 722)
flagemblem <guild_id>;

// Spawn a castle guardian
guardian "<map>", <x>, <y>, "<display_name>", <mob_id>{, "<event_label>"{, <index>}};

// Get guardian info
guardianinfo("<map>", <index>, <type>)
// type: 0 = visible, 1 = max HP, 2 = current HP
```

---

## WoE NPC Labels

These labels fire automatically on ALL NPCs during WoE events:

```c
OnAgitStart:    // WoE FE begins
OnAgitEnd:      // WoE FE ends
OnAgitStart2:   // WoE SE begins
OnAgitEnd2:     // WoE SE ends
OnAgitStart3:   // WoE TE begins
OnAgitEnd3:     // WoE TE ends
```

---

## Automated WoE Schedule (controller NPC)

```
// Place anywhere, no map position needed
-	script	WoE_Controller	FAKE_NPC,{
	end;

// WoE FE: Monday and Thursday 20:00-22:00
OnClock2000:
	if (gettime(GETTIME_DAYOFWEEK) == 1 || gettime(GETTIME_DAYOFWEEK) == 4) {
		agitstart;
		announce "War of Emperium has begun!", bc_all;
	}
	end;

OnClock2200:
	if (agitcheck()) {
		agitend;
		announce "War of Emperium has ended!", bc_all;
	}
	end;

// WoE SE: Saturday 19:00-21:00
OnClock1900:
	if (gettime(GETTIME_DAYOFWEEK) == 6) {
		agitstart2;
		announce "War of Emperium SE has begun!", bc_all;
	}
	end;

OnClock2100:
	if (agitcheck2()) {
		agitend2;
		announce "War of Emperium SE has ended!", bc_all;
	}
	end;
}
```

---

## Castle Configuration (`db/re/castle_db.yml`)

```yaml
- CastleId: 0
  MapName: prtg_cas01
  CastleName: Hohenschwangau
  NpcName: Steward#prtg01
  GuildFlag: flagyard#prtg01
  Guardian1: guardian#prtg01_01
  Guardian2: guardian#prtg01_02
  # ... up to 8 guardians
  Treasure1: Treasure#prtg01_01
  # ... up to 4 treasure boxes
```

---

## Guild Info in Scripts

```c
getguildname(<guild_id>)          // Guild name string
getguildmaster(<guild_id>)        // Master character name
getguildmasterid(<guild_id>)      // Master char ID
getcharid(CHAR_ID_GUILD)          // Current player's guild ID
getcharid(CHAR_ID_PARTY)          // Current player's party ID

// Check if player owns a castle
// (check via variable or NPC ownership tracking)
```

---

## Battleground Mode

```c
// Enable battleground on a map
setmapflag "<map>", mf_battleground;         // Standard BG
setmapflag "<map>", mf_battleground, 2;      // With scoreboard

// Remove
removemapflag "<map>", mf_battleground;

// Battleground queue (bg_* commands)
bg_create_team <map>, <x>, <y>, <logineventlabel>, <dieventlabel>;
// Returns team ID

bg_join <team_id>, <map>, <x>, <y>;          // Add player to BG team
bg_leave {<team_id>};                         // Remove player from team

bg_warp <team_id>, "<map>", <x>, <y>;        // Warp whole team
bg_team_setxy <team_id>, <x>, <y>;           // Reposition team

bg_get_data(<team_id>, <type>)               // Get team info
// type: 0 = player count

bg_reward <team_id>, <fame>, <bg_points>, <item_id>, <item_amount>;
// Reward entire team

bg_destroy <team_id>;                         // Destroy team
```

---

## Map Flags for WoE/GvG

```c
// Typical WoE castle setup
setmapflag "prtg_cas01", mf_gvg_castle;
setmapflag "prtg_cas01", mf_noteleport;
setmapflag "prtg_cas01", mf_nopenalty;
setmapflag "prtg_cas01", mf_noexp;
setmapflag "prtg_cas01", mf_partylock;
setmapflag "prtg_cas01", mf_guildlock;
setmapflag "prtg_cas01", mf_nobranch;

// GvG dungeon
setmapflag "gld_dun01", mf_gvg_dungeon;
setmapflag "gld_dun01", mf_noteleport;
```
