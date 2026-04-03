# rAthena YAML Database Reference

All game data files are in `db/re/` (renewal) or `db/pre-re/` (pre-renewal).

---

## Item Database

### Files
| File | Contents |
|------|----------|
| `item_db_equip.yml` | Weapons, armor, ammo, cards |
| `item_db_usable.yml` | Consumables, usable items |
| `item_db_etc.yml` | Misc items, drops, materials, quest items |
| `item_combos.yml` | Equipment combo bonuses |
| `item_group_db.yml` | Random item drop groups |
| `item_packages.yml` | Client-selectable packages |
| `item_enchant.yml` | Enchanting slot rules |
| `item_reform.yml` | Upgrade/reform paths |

### Item Entry Structure

```yaml
- Id: 2102
  AegisName: Shield_Of_Athena
  Name: Shield of Athena
  Type: Armor
  Buy: 20000
  Weight: 500
  Defense: 90
  Slots: 1
  Jobs:
    All: true
  Classes:
    All: true
  Gender: Both
  Locations:
    Shield: true
  WeaponLevel: 1
  ArmorLevel: 1
  EquipLevelMin: 50
  Refineable: true
  Script: |
    bonus bMdef, 10;
    bonus bDef, 5;
  EquipScript: |
    sc_start SC_BLESSING, 600000, 10;
  UnEquipScript: |
    sc_end SC_BLESSING;
  Flags:
    BuyingStore: true
    DeadBranch: false
    BindOnEquip: false
    DropAnnounce: false
    NoConsume: false
    DropEffect: DROPEFFECT_NONE
  Delay:
    Duration: 0
    Status: ""
  Trade:
    NoDrop: false
    NoTrade: false
    TradePartner: false
    NoSell: false
    NoCart: false
    NoStorage: false
    NoGuildStorage: false
    NoMail: false
    NoAuction: false
  Stack:
    Amount: 0
    Inventory: false
    Cart: false
    Storage: false
    GuildStorage: false
```

### Bonus Script Examples

```c
// Stats
bonus bStr, 10;         // +10 STR
bonus bAgi, 5;          // +5 AGI
bonus bVit, 8;          // +8 VIT
bonus bInt, 10;         // +10 INT
bonus bDex, 7;          // +7 DEX
bonus bLuk, 3;          // +3 LUK

// HP/SP
bonus bMaxHp, 500;      // +500 MaxHP
bonus bMaxSp, 100;      // +100 MaxSP
bonus bMaxHpRate, 10;   // +10% MaxHP
bonus bMaxSpRate, 5;    // +5% MaxSP

// Defense
bonus bDef, 10;         // +10 DEF
bonus bMdef, 5;         // +5 MDEF
bonus bDefRate, 10;     // +10% DEF

// Attack
bonus bAtk, 50;         // +50 ATK
bonus bMatk, 30;        // +30 MATK
bonus bAtkRate, 5;      // +5% ATK

// Elements
bonus bAtkEle, Ele_Fire;    // Fire element attack
bonus bDefEle, Ele_Water;   // Water element defense
bonus bMagicAtkEle, Ele_Holy; // Holy magic attacks

// Resistances
bonus2 bSubEle, Ele_Fire, 10;    // +10% resist fire
bonus2 bSubRace, RC_Undead, 15;  // +15% resist undead

// Critical
bonus bCritical, 10;    // +10 CRIT
bonus bCritAtkRate, 5;  // +5% crit damage

// Speed
bonus bAspd, 2;         // +2 ASPD
bonus bAspdRate, 5;     // +5% ASPD
bonus bSpeed, -25;      // Move speed (negative = faster)
```

---

## Monster Database

### File: `mob_db.yml`

```yaml
- Id: 1002
  AegisName: PORING
  Name: Poring
  JapaneseName: Poring
  Level: 1
  Hp: 60
  BaseExp: 2
  JobExp: 1
  MvpExp: 0
  Attack: 7
  Attack2: 10
  Defense: 0
  MagicDefense: 5
  Str: 1
  Agi: 1
  Vit: 0
  Int: 0
  Dex: 6
  Luk: 30
  AttackRange: 1
  SkillRange: 10
  ChaseRange: 12
  Size: Small
  Race: Plant
  RaceGroups:
    Poring: true
  Element: Water
  ElementLevel: 1
  WalkSpeed: 400
  AttackDelay: 1872
  AttackMotion: 672
  ClientAttackMotion: 288
  DamageMotion: 480
  DamageTakenRate: 100
  Ai: 02
  Class: Normal
  Modes:
    CanMove: true
    CanAttack: false
  MvpDrops: []
  Drops:
    - Item: Apple
      Rate: 1500      # 15% (10000 = 100%)
      StealProtected: false
      RandomOptionGroup: ""
      Index: 0
    - Item: Jellopy
      Rate: 7000      # 70%
```

### Mob AI Values
- `01` = Passive (won't attack first)
- `02` = Passive, random walk
- `04` = Aggressive, attacks on sight
- `05` = Aggressive, large chase range
- `09` = Boss behavior
- `20` = MVP behavior

### Mob Modes
```yaml
Modes:
  CanMove: true
  CanAttack: true
  NoCast: false
  CastSensor: true       # Chases casting players
  BossType: false        # MVP/mini-boss behavior
  Plant: false           # HP always 1
  NoKnockback: false
  TeleportBlock: false   # Prevents escape via teleport
  IgnoreMagic: false
  IgnoreMelee: false
  IgnoreMisc: false
  IgnoreRanged: false
  Detector: false        # Detects hidden players
  ChangeTargetMelee: false
  ChangeTargetChase: false
  TargetWeak: false
  RandomTarget: false
```

---

## Skill Database

### File: `skill_db.yml`

```yaml
- Id: 1
  Name: NV_BASIC
  Description: Basic Skill
  MaxLevel: 9
  Type: Passive
  TargetType: Self

- Id: 28
  Name: SM_BASH
  Description: Bash
  MaxLevel: 10
  Type: Attack
  TargetType: Attack
  DamageFlags:
    Splash: false
  Flags:
    IgnoreNpc: false
  Range:
    - Level: 1
      Size: 1
  Hit: Single
  HitCount: 1
  Element: Weapon
  SplashArea: 0
  ActiveInstance: 0
  CastTime:
    - Level: 1
      Time: 0
  AfterCastActDelay:
    - Level: 1
      Time: 1000
  SkillData1:
    - Level: 1
      Time: 0
  FixedCastTime:
    - Level: 1
      Time: 0
  CastTimeFlags:
    IgnoreDex: false
    IgnoreStatus: false
    IgnoreItemBonus: false
  CastDelayFlags:
    IgnoreDex: false
    IgnoreStatus: false
    IgnoreItemBonus: false
  Requires:
    HpCost:
      - Level: 1
        Amount: 0
    SpCost:
      - Level: 1
        Amount: 10
  Unit:
    Id: 0
    Layout: 0
    Range: 0
    Interval: 0
    Target: All
    Flag: 0
```

---

## Quest Database

### File: `quest_db.yml`

```yaml
- Id: 1001
  Title: Find the Poring
  Time: 3600      # Time limit in seconds (0 = no limit)
  Targets:
    - Mob: PORING
      Count: 5
  Drops:
    - Item: Apple
      Count: 1
      Mob: PORING    # Optional: only from this mob
```

---

## Constants Reference

### Element IDs
```
Ele_Neutral, Ele_Water, Ele_Earth, Ele_Fire, Ele_Wind,
Ele_Poison, Ele_Holy, Ele_Dark, Ele_Ghost, Ele_Undead
```

### Race IDs
```
RC_Formless, RC_Undead, RC_Brute, RC_Plant, RC_Insect,
RC_Fish, RC_Demon, RC_DemiHuman, RC_Angel, RC_Dragon, RC_Player
```

### Size IDs
```
Size_Small, Size_Medium, Size_Large
```

### Item Type
```
Healing, Usable, Etc, Armor, Weapon, Card,
PetEgg, PetArmor, Ammo, UsableWithDelayed,
Shadow, UsableWithDelayed2, Cannonball
```

### Equipment Locations
```
Head_Top, Head_Mid, Head_Low, Armor, Right_Hand,
Left_Hand, Garment, Shoes, Right_Accessory, Left_Accessory,
Costume_Head_Top, Costume_Head_Mid, Costume_Head_Low,
Costume_Garment, Shadow_Armor, Shadow_Weapon,
Shadow_Shield, Shadow_Shoes, Shadow_Right_Accessory,
Shadow_Left_Accessory, Both_Hand, Both_Accessory
```

---

## mob_avail.yml — Sprite Override (player sprite on a monster)

Remaps a monster's visual appearance to a player job sprite, without any client modification.
File: `db/mob_avail.yml`

```yaml
- Mob: FAKE_KNIGHT          # AegisName from mob_db.yml
  Sprite: JOB_LORD_KNIGHT   # Player job sprite to use
  Sex: Male                 # Male / Female
  HairStyle: 5              # Hair style number (1-30+)
  HairColor: 3              # Hair color index (0-8)
  ClothColor: 2             # Cloth color index (0-8)
  Weapon: 118               # Equipment View ID for weapon (0 = none)
  Shield: 0                 # Equipment View ID for shield (0 = none)
  HeadTop: 5                # Head accessory top view ID (0 = none)
  HeadMid: 0                # Head accessory mid
  HeadLow: 0                # Head accessory bottom
```

**Important:** `Weapon`/`Shield`/`HeadTop/Mid/Low` use **ViewSprite IDs** from `item_db_equip.yml`,
not item IDs.

Available job sprite constants: `JOB_NOVICE`, `JOB_SWORDMAN`, `JOB_KNIGHT`, `JOB_LORD_KNIGHT`,
`JOB_WIZARD`, `JOB_HIGH_WIZARD`, `JOB_PRIEST`, `JOB_HIGH_PRIEST`, `JOB_ASSASSIN`,
`JOB_ASSASSIN_CROSS`, `JOB_HUNTER`, `JOB_SNIPER`, `JOB_BLACKSMITH`, `JOB_WHITESMITH`, etc.

Reload without restart: `@reloadmobdb`

---

## Hot-Reload Commands (GM)

```
@reloaditemdb      Reload item database
@reloadmobdb       Reload monster database
@reloadskilldb     Reload skill database
@reloadscript      Reload all NPC scripts
@reloadbattleconf  Reload battle configuration
@reloadstatusdb    Reload status database
@reloadquestdb     Reload quest database
```
