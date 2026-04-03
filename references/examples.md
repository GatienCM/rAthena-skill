# rAthena Complete NPC Examples

Ready-to-use NPC scripts covering the most common use cases.

---

## 1. Quest NPC (fetch + kill quest)

```
prontera,155,180,5	script	Quest Giver::QuestGiver01	4_M_BIOLOGIST,{
	// Show current quest state
	switch (checkquest(1001)) {
		case QUEST_INACTIVE:
			mes "[Quest Giver]";
			mes "I need your help! Bring me 10 Jellopy and slay 5 Porings.";
			next;
			if (select("Accept quest", "Maybe later") == 2) close;
			setquest 1001;
			mes "Quest accepted! Good luck.";
			close;

		case QUEST_ACTIVE:
			mes "[Quest Giver]";
			if (countitem(909) < 10) {
				mes "You still need " + (10 - countitem(909)) + " more Jellopy.";
				mes "And kill " + (5 - checkquest(1001, HUNTING)) + " more Porings.";
			} else {
				mes "Turn in your quest?";
				next;
				if (select("Yes", "Not yet") == 2) close;
				delitem 909, 10;
				completequest 1001;
				getitem 512, 5;      // 5 Red Potions reward
				getexp 500, 300;     // Base + Job EXP
				mes "Thank you! Here's your reward.";
			}
			close;

		case QUEST_COMPLETE:
			mes "[Quest Giver]";
			mes "You've already completed this quest. Thank you!";
			close;
	}
	close;
}
```

**quest_db.yml entry:**
```yaml
- Id: 1001
  Title: Poring Problem
  Targets:
    - Mob: PORING
      Count: 5
  Drops:
    - Item: Jellopy
      Count: 10
      Mob: PORING
```

---

## 2. Daily Reward NPC (24h cooldown)

```
prontera,150,175,5	script	Daily Reward::DailyReward	4_F_KAFRA1,{
	.@cooldown = 86400;  // 24 hours

	if (@daily_last_claim > 0 && gettimetick(2) - @daily_last_claim < .@cooldown) {
		.@remaining = .@cooldown - (gettimetick(2) - @daily_last_claim);
		.@hours = .@remaining / 3600;
		.@minutes = (.@remaining % 3600) / 60;
		mes "[Daily Reward]";
		mes "You already claimed today's reward.";
		mes "Come back in: ^FF0000" + .@hours + "h " + .@minutes + "m^000000";
		close;
	}

	mes "[Daily Reward]";
	mes "Welcome! Choose your daily reward:";
	next;

	.@choice = select(
		"100,000 Zeny",
		"5x Yggdrasil Berry",
		"Experience Scroll (+10% Base EXP)"
	);

	set @daily_last_claim, gettimetick(2);

	switch (.@choice) {
		case 1:
			set Zeny, Zeny + 100000;
			mes "You received 100,000 Zeny!";
			break;
		case 2:
			getitem 607, 5;
			mes "You received 5x Yggdrasil Berry!";
			break;
		case 3:
			// +10% base EXP for 1 hour via bonus_script
			bonus_script "{ bonus bBaseExpRate, 10; }", 3600000, 8;
			mes "EXP boost active for 1 hour!";
			break;
	}
	close;
}
```

---

## 3. Custom Point / VIP Shop

```
prontera,160,180,5	script	Point Shop::PointShop01	4_F_KAFRA1,{
	mes "[Point Shop]";
	mes "Your points: ^0055FF" + #MyPoints + "^000000";
	next;

OnMenu:
	.@choice = select(
		"[100 pts] Red Potion x10",
		"[500 pts] Yggdrasil Berry x3",
		"[1000 pts] Old Blue Box",
		"[2000 pts] Costume Hat",
		"Close"
	);

	if (.@choice == 5) close;

	setarray .@costs[1], 100, 500, 1000, 2000;
	setarray .@items[1], 501, 607, 617, 5006;  // item IDs
	setarray .@amounts[1], 10, 3, 1, 1;

	if (#MyPoints < .@costs[.@choice]) {
		mes "Not enough points! You need " + .@costs[.@choice] + " pts.";
		next;
		goto OnMenu;
	}

	set #MyPoints, #MyPoints - .@costs[.@choice];
	getitem .@items[.@choice], .@amounts[.@choice];
	mes "Purchase successful! Points left: ^0055FF" + #MyPoints + "^000000";
	next;
	goto OnMenu;
}

// GM command NPC to add points (separate, GM-only)
prontera,158,180,5	script	Points GM::PointsGM	FAKE_NPC,{
	end;

OnCall:
	// Call via: donpcevent "Points GM::OnCall"
	// after setting target and amount via variables
	end;
}
```

---

## 4. Timed Event Boss

```
// Controller NPC — no map position
-	script	Event Controller::EventCtrl	FAKE_NPC,{
	end;

// Every day at 20:00
OnClock2000:
	if ($event_active) end;  // Already running

	set $event_active, 1;
	announce "A powerful boss has appeared in Prontera!", bc_all|bc_yellow;

	// Spawn boss with kill event
	monster "prontera", 155, 155, "Event Boss", 1272, 1, "Event Controller::OnBossDead";

	// Start 30-minute timer
	initnpctimer;
	end;

OnTimer1800000:  // 30 minutes
	if (!$event_active) end;
	announce "The event boss has fled! Better luck next time.", bc_all;
	killmonster "prontera", "Event Controller::OnBossDead";
	set $event_active, 0;
	stopnpctimer;
	end;

OnBossDead:
	set $event_active, 0;
	stopnpctimer;
	announce "The Event Boss has been slain! Congratulations!", bc_all|bc_yellow;

	// Reward everyone on the map
	mapwarp "prontera", "prontera", 155, 155;  // Keep them in place
	// Use areamonster or loop to reward
	end;
}
```

---

## 5. Warp Portal NPC (multi-destination)

```
prontera,148,175,5	script	Warp NPC::WarpNPC01	4_F_KAFRA1,{
	mes "[Kafra Employee]";
	mes "Where would you like to go?";
	next;

	.@choice = select(
		"Morroc (200z)",
		"Geffen (200z)",
		"Payon (200z)",
		"Alberta (500z)",
		"Izlude (Free)",
		"Cancel"
	);

	if (.@choice == 6) close;

	setarray .@maps$[1], "morocc", "geffen", "payon", "alberta", "izlude";
	setarray .@x[1],     156,      119,      233,     100,       128;
	setarray .@y[1],     93,       59,       223,     143,       146;
	setarray .@cost[1],  200,      200,      200,     500,       0;

	if (Zeny < .@cost[.@choice]) {
		mes "You need " + .@cost[.@choice] + " zeny.";
		close;
	}

	set Zeny, Zeny - .@cost[.@choice];
	warp .@maps$[.@choice], .@x[.@choice], .@y[.@choice];
	end;
}
```

---

## 6. Refine NPC (weapon upgrade)

```
prontera,152,180,5	script	Refiner::Refiner01	4_M_BIOLOGIST,{
	mes "[Blacksmith]";
	mes "I can refine your equipment. The higher the refine, the greater the risk!";
	next;

	// Get equipped weapon
	.@item_id = getequipid(EQI_HAND_R);
	if (.@item_id == -1) {
		mes "You don't have a weapon equipped!";
		close;
	}

	.@refine_lv = getequiprefinerycnt(EQI_HAND_R);

	if (.@refine_lv >= 10) {
		mes "Your weapon is already at maximum refine (+10).";
		close;
	}

	// Cost scales with refine level
	.@cost = 5000 * (.@refine_lv + 1);

	mes "Current weapon: ^0055FF" + getitemname(.@item_id) + "^000000 (+" + .@refine_lv + ")";
	mes "Refine cost: ^FF0000" + .@cost + " Zeny^000000";
	mes "Success chance: ^00AA00" + (100 - .@refine_lv * 10) + "%^000000";
	next;

	if (select("Refine!", "Cancel") == 2) close;

	if (Zeny < .@cost) {
		mes "Not enough zeny!";
		close;
	}

	set Zeny, Zeny - .@cost;

	// Success rate: 100% at +0, drops 10% per level
	if (rand(100) < (100 - .@refine_lv * 10)) {
		successrefitem EQI_HAND_R;
		mes "^00AA00Success!^000000 Your weapon is now +" + (.@refine_lv + 1) + "!";
	} else {
		failedrefitem EQI_HAND_R;
		mes "^FF0000Failed!^000000 The refine broke your weapon...";
	}
	close;
}
```

---

## 7. Login / Logout Event NPC

```
-	script	Login Events::LoginEvents	FAKE_NPC,{
	end;

OnPCLoginEvent:
	// Give daily login bonus
	if (gettimetick(2) - #last_login > 86400) {
		set #login_streak, #login_streak + 1;
		set #last_login, gettimetick(2);

		// Reward based on streak
		if (#login_streak >= 7) {
			getitem 617, 1;  // Weekly bonus
			set #login_streak, 0;
			message strcharinfo(PC_NAME), "Weekly login bonus: Old Blue Box!";
		} else {
			getitem 501, 5;  // Daily bonus
			message strcharinfo(PC_NAME), "Day " + #login_streak + " login bonus: 5x Red Potion!";
		}
	}

	// Apply VIP buff if player has VIP status
	if (#vip_expiry > gettimetick(2)) {
		bonus_script "{ bonus bBaseExpRate, 50; bonus bJobExpRate, 50; }", 86400000, 8;
	}
	end;

OnPCLogoutEvent:
	// Log logout time
	set #last_logout, gettimetick(2);
	end;
}
```

---

## 8. NPC Tips & Conventions

```c
// Always use unique NPC names with ::UniqueName to avoid conflicts
prontera,100,100,4	script	My NPC::MyNPC_UniqueID	4_F_KAFRA1,{

// Use strnpcinfo() to get NPC info
strnpcinfo(NPC_NAME)    // Display name ("My NPC")
strnpcinfo(NPC_SRCFILE) // Source file path
strnpcinfo(NPC_MAP)     // Map the NPC is on

// Prefer close2 + end when you need to do things after dialog
mes "Warping you now...";
close2;
warp "prontera", 155, 180;
end;

// Use playerattached() to safely check for player context in timers
OnTimer5000:
	if (!playerattached()) end;
	mes "Timer fired!";
	end;
}
```
