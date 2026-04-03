# rAthena Map Flags Reference

## Commands

```c
setmapflag "<map>", <flag>{, <value>};
removemapflag "<map>", <flag>{, <value>};
getmapflag("<map>", <flag>{, <type>})   // Returns 0/1 or value
```

---

## Teleportation & Movement

| Flag | Description |
|------|-------------|
| `mf_noteleport` | Blocks ALL teleportation (items, skills, scripts) |
| `mf_nowarp` | Prevents warping FROM this map (@warp, scripts) |
| `mf_nowarpto` | Prevents warping TO this map |
| `mf_noreturn` | Blocks map-warping items (Butterfly Wings, etc.) |
| `mf_nogo` | Blocks @go command |
| `mf_nomemo` | Disables /memo and marriage warp |
| `mf_nosave <map>,<x>,<y>` | Auto-save to alternate map on logout |
| `mf_monster_noteleport` | Monsters cannot teleport |

## Combat & PvP

| Flag | Description |
|------|-------------|
| `mf_pvp` | Enable PvP mode |
| `mf_pvp_noparty` | Party members can attack each other |
| `mf_pvp_noguild` | Guild members can attack each other |
| `mf_pvp_nocalcrank` | No PvP ranking calculation |
| `mf_pvp_nightmaredrop <id>,<type>,<rate>` | Drop items on death (type: 1=inventory, 2=equip, 3=both) |
| `mf_gvg` | Enable GvG mode |
| `mf_gvg_noparty` | GvG without party restriction |
| `mf_gvg_castle` | Castle GvG map |
| `mf_gvg_dungeon` | GvG dungeon map |
| `mf_gvg_te` | Tactical Emperium GvG |
| `mf_gvg_te_castle` | TE Castle map |
| `mf_battleground {<type>}` | Battleground mode (type 2 = scoreboard) |
| `mf_nolockon` | No free-form PvP (must Shift+click) |
| `mf_allowks` | Allow kill-stealing |

## Skills & Items

| Flag | Description |
|------|-------------|
| `mf_noskill` | All skills disabled |
| `mf_noicewall` | WZ_ICEWALL disabled |
| `mf_nobranch` | Monster-spawning items disabled (Dead Branch, etc.) |
| `mf_noitemconsumption` | Players cannot use items |
| `mf_restricted <zone>` | Items/skills disabled by zone (1-8, see `conf/battle/skill.conf`) |
| `mf_nosunmoonstarmiracle` | Star Gladiator miracles disabled |
| `mf_nopetcapture` | Pet capture disabled |
| `mf_skill_damage <skill>,<caster>,<vs_pc>,<vs_mob>,<vs_boss>,<vs_other>` | Adjust skill damage % |
| `mf_skill_duration <skill>,<percentage>` | Adjust trap/skill duration |

## Economy & Social

| Flag | Description |
|------|-------------|
| `mf_notrade` | Trading disabled |
| `mf_nodrop` | Item drops disabled |
| `mf_noloot` | All monster drops disabled |
| `mf_nomobloot` | Normal monster drops disabled |
| `mf_nomvploot` | MVP drops disabled |
| `mf_novending` | Shop creation (MC_VENDING) disabled |
| `mf_nobuyingstore` | Buying stores disabled |
| `mf_nousecart` | Cart usage blocked |
| `mf_nochat` | Chatroom creation disabled |
| `mf_nobank` | Bank access disabled |
| `mf_norodex` | RODex mail disabled |
| `mf_partylock` | Prevent party changes on map |
| `mf_guildlock` | Prevent guild changes on map |

## Experience & Penalties

| Flag | Value | Description |
|------|-------|-------------|
| `mf_noexp` | — | No experience gain |
| `mf_nobaseexp` | — | No base EXP |
| `mf_nojobexp` | — | No job EXP |
| `mf_nopenalty` | — | No death penalties |
| `mf_noexppenalty` | — | No EXP death penalty |
| `mf_nozenypenalty` | — | No Zeny death penalty |
| `mf_norenewaldroppenalty` | — | No level-diff drop penalty |
| `mf_norenewalexppenalty` | — | No level-diff EXP penalty |
| `mf_bexp <rate>` | 0-10000 | Base EXP rate (100 = 1x) |
| `mf_jexp <rate>` | 0-10000 | Job EXP rate (100 = 1x) |

## Visual & Atmosphere

| Flag | Description |
|------|-------------|
| `mf_clouds` | Cloud weather effect |
| `mf_clouds2` | Cloud weather effect variant |
| `mf_fireworks` | Fireworks effect |
| `mf_fog` | Fog effect |
| `mf_leaves` | Falling leaves effect |
| `mf_sakura` | Sakura petals effect |
| `mf_snow` | Snow effect |
| `mf_nightenabled` | Night mode visuals |
| `mf_forcemineffect` | Force minimal skill effects |
| `mf_nocostume` | Hide costume sprites |
| `mf_hidemobhpbar` | Hide monster HP bars |
| `mf_notomb` | Disable MVP tombs |

## Miscellaneous

| Flag | Description |
|------|-------------|
| `mf_town` | Town map (mail access, no kill-steal) |
| `mf_reset` | Allows Neuralizer usage |
| `mf_loadevent` | Triggers `OnPCLoadMapEvent` label on map entry |
| `mf_autotrade` | Enables @autotrade on this map |
| `mf_nocommand <group_level>` | @commands disabled below group level |
| `mf_nomapchannelautojoin` | No auto-join map channel |
| `mf_specialpopup <popup_id>` | Show popup on map entry |
| `mf_invincible_time <ms>` | Post-login invincibility duration |

---

## Practical Examples

```c
// Dungeon: no escape, no penalties, aggressive feel
setmapflag "my_dungeon", mf_noteleport;
setmapflag "my_dungeon", mf_noreturn;
setmapflag "my_dungeon", mf_nopenalty;
setmapflag "my_dungeon", mf_nobranch;

// PvP arena: full PvP, no party immunity
setmapflag "pvp_arena", mf_pvp;
setmapflag "pvp_arena", mf_pvp_noparty;
setmapflag "pvp_arena", mf_pvp_noguild;
setmapflag "pvp_arena", mf_noexp;
setmapflag "pvp_arena", mf_nopenalty;

// Event map: double EXP, custom drops
setmapflag "event_map", mf_bexp, 200;
setmapflag "event_map", mf_jexp, 200;
setmapflag "event_map", mf_snow;

// Safe town: no PvP, no drops
setmapflag "safe_town", mf_town;
setmapflag "safe_town", mf_nodrop;
setmapflag "safe_town", mf_noteleport;

// Check and remove a flag
if (getmapflag("prontera", mf_pvp)) {
    removemapflag "prontera", mf_pvp;
}
```

---

## Map Flag Files (npc/mapflag/)

rAthena loads map flags from script files at startup:
```
npc/mapflag/pvp.txt
npc/mapflag/gvg.txt
npc/mapflag/skill.txt
npc/mapflag/town.txt
... etc.
```

To add permanent map flags without scripting, add a line to the relevant file:
```
prontera    mapflag    town
my_map      mapflag    noteleport
my_map      mapflag    bexp    200
```
