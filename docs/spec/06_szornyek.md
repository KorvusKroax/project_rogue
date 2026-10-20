# 6. Szörnyek

## 6.1 Típusok

A 26 szörnytípus adatai. A különleges képességeket (spec) a [6.7](#67-speciális-támadások-sikeres-találat-után-ha-nincs-iscanc) írja le.

| | név | carry | flags | lvl | arm | exp | dmg | spec |
|---|---|---|---|---|---|---|---|---|
| A | aquator | 0 | MEAN | 5 | 2 | 20 | 0x0/0x0 | rozsda |
| B | bat | 0 | FLY | 1 | 3 | 1 | 1x2 | kaotikus 50% |
| C | centaur | 15 | – | 4 | 4 | 17 | 1x2/1x5/1x5 | |
| D | dragon | 100 | MEAN | 10 | −1 | 5000 | 1x8/1x8/3x10 | tűz |
| E | emu | 0 | MEAN | 1 | 7 | 2 | 1x2 | |
| F | venus flytrap | 0 | MEAN | 8 | 3 | 80 | %%%x0 | megfog |
| G | griffin | 20 | MEAN,FLY,REGEN | 13 | 2 | 2000 | 4x3/3x5 | |
| H | hobgoblin | 0 | MEAN | 1 | 5 | 3 | 1x8 | |
| I | ice monster | 0 | – | 1 | 9 | 5 | 0x0 | fagyaszt |
| J | jabberwock | 70 | – | 15 | 6 | 3000 | 2x12/2x4 | |
| K | kestrel | 0 | MEAN,FLY | 1 | 7 | 1 | 1x4 | |
| L | leprechaun | 0 | – | 3 | 8 | 10 | 1x1 | aranylopás |
| M | medusa | 40 | MEAN | 8 | 2 | 200 | 3x4/3x4/2x5 | zavaró pillantás |
| N | nymph | 100 | – | 3 | 9 | 37 | 0x0 | tárgylopás |
| O | orc | 15 | GREED | 1 | 6 | 5 | 1x8 | |
| P | phantom | 0 | INVIS | 8 | 3 | 120 | 4x4 | kaotikus 20% |
| Q | quagga | 0 | MEAN | 3 | 3 | 15 | 1x5/1x5 | |
| R | rattlesnake | 0 | MEAN | 2 | 3 | 9 | 1x6 | méreg |
| S | snake | 0 | MEAN | 1 | 5 | 2 | 1x3 | |
| T | troll | 50 | REGEN,MEAN | 6 | 4 | 120 | 1x8/1x8/2x6 | |
| U | black unicorn | 0 | MEAN | 7 | −2 | 190 | 1x9/1x9/2x9 | |
| V | vampire | 20 | REGEN,MEAN | 8 | 1 | 350 | 1x10 | HP-szívás |
| W | wraith | 0 | – | 5 | 4 | 55 | 1x6 | szintszívás |
| X | xeroc | 30 | – | 7 | 7 | 100 | 4x4 | álcázás |
| Y | yeti | 30 | – | 4 | 6 | 50 | 1x6/1x6 | |
| Z | zombie | 0 | MEAN | 2 | 8 | 6 | 1x8 | |

Minden szörny `str = 10`.

## 6.2 Generálás
```
INITIAL = "KEBSHIROZLCQANYFTWPXUMVGJD"
WANDER  = "KEBSH0ROZ0CQA0Y0TWP0UMVGJ0"  // 0 = nem lehet vándorló

new_monster(depth, type_list):  // depth: a dungeon-szint; type_list: INITIAL vagy WANDER
  loop:
    index = depth + rnd(10) - 6               // a mélység körüli véletlen pozíció a listában
    if index < 0:  index = rnd(5)
    if index > 25: index = rnd(5) + 21
    if type_list[index] != 0: break           // 0: ezen a helyen nincs vándorló
  type = TYPES[type_list[index]]
  depth_bonus = max(depth - AMULET_LEVEL, 0)  // az amulett szintje alatt erősebbek a szörnyek
  monster.lvl = type.lvl + depth_bonus
  monster.maxhp = monster.hpt = ROLL(monster.lvl, 8)
  monster.arm = type.arm - depth_bonus
  monster.dmg = type.dmg; monster.str = type.str; monster.flags = type.flags
  monster.exp = type.exp + depth_bonus*10 + expmod(monster.lvl, monster.maxhp)
  if depth > 29: monster.flags |= ISHASTE
  monster.turn = true                         // lassítva az első körben lép
  monster.oldch = a mező jelenlegi képernyőjele
  monster a szörnylista ELEJÉRE kerül         // a runners így a legújabbal kezd
  if aggravate ring worn: runto(monster)      // ISRUN, ISHELD le, dest = find_dest
  monster.disguise = type.letter
  if type.letter == 'X': monster.disguise = rnd_thing()
       // egyenletesen: ! ? = / : ) ] % *   és a 26. szinttől a , (amulett) is
       // (POTION, SCROLL, RING, STICK, FOOD, WEAPON, ARMOR, STAIRS, GOLD, [AMULET])

expmod(lvl, hp):  // az XP-érték HP-tól függő része
  bonus = (lvl == 1) ? hp/8 : hp/6
  if lvl > 9: bonus *= 20 else if lvl > 6: bonus *= 4
  return bonus
```

**Vándorló szörny démon:**
```
// induláskor: időzítő spread(70) (AFTER fázis); lejártakor elindul a rollwand démon (BEFORE fázis)
rollwand():  // BEFORE fázis, körönként egyszer
  if ++wander_check_tick >= 4:  // csak minden 4. körben dob
     if ROLL(1,6) == 4:
        wanderer()
        rollwand démon le; új időzítő spread(70), ezúttal a BEFORE fázisban
     wander_check_tick = 0

wanderer():
  hely: find_floor(null, monster), ismételve amíg nem a játékos szobája
  monster = new_monster(level, WANDER)
  if SEEMONST: monster kirajzolása inverzen (hallucinálva véletlen betűvel)
  runto(monster)  // ISRUN; dest = find_dest(monster) – általában a játékos
  nem kap tárgyat
```

## 6.3 Kezdő tárgy
```
give_item(monster):
  if level >= max_level and rnd(100) < monster.carry: monster.pack += new_item()
```

## 6.4 Ébresztés

**Mikor hívódik a `wake_monster`:**
1. **Szobába belépéskor és szoba elhagyásakor** (`enter_room` / `leave_room` → `door_open`): a szoba **összes** szörnyére, világos és sötét szobában egyaránt (fantom szobánál nem).
2. **Minden játékos-akció előtt** (`look(TRUE)` a parancsciklusban): a játékos körüli **8 szomszédos** mezőn álló, látható szörnyekre. Átlós szomszéd csak akkor, ha nem sarkon át esik.

```
wake_monster(monster):
  // 1. aljas szörny
  if !monster.ISRUN and rnd(3) != 0 and monster.ISMEAN and !monster.ISHELD
     and !wearing(STEALTH) and !player.ISLEVIT:
       monster.dest = player; monster.flags |= ISRUN
  // 2. Medusa pillantása – minden hívásnál próbálkozik, amíg el nem sütötte (ISFOUND)
  if monster.type == 'M' and !player.ISBLIND and !player.ISHALU
     and !monster.ISFOUND and !monster.ISCANC and monster.ISRUN:
       if (a JÁTÉKOS szobája világos) or dist2(monster, player) < 3:
          monster.flags |= ISFOUND                      // a mentődobás előtt → csak egy esély
          if !save_magic():
             ISHUH; ha már zavarodott: az időzítő +spread(20), különben új időzítő spread(20)
             név(monster) + "'s gaze has confused you"  // pl. "the medusa's gaze …"; ha név(monster) == "it": "its gaze …"
  // 3. kapzsi szörny – nincs 2/3 esély, a stealth és a lebegés sem védi ki, ISHELD-et sem nézi
  if monster.ISGREED and !monster.ISRUN:
       monster.flags |= ISRUN
       monster.dest = (a JÁTÉKOS szobájában van arany) ? annak helye : player

runto(monster):  // a "kényszerített" ébresztés
  monster.flags |= ISRUN; monster.flags &= ~ISHELD; monster.dest = choose_dest(monster)
```

**Nem ISMEAN és nem ISGREED szörny** (B, C, I, J, L, N, P, W, X, Y) a `wake_monster`-től **soha nem ébred fel**, akármilyen közel megy hozzá a játékos, vagy akárhányszor lép be a szobájába. Csak `runto` indítja el, ezek pedig:
- a játékos megtámadja (közelharc, dobott tárgy – `fight()` eleje);
- pálca/bot hat rá (haste/slow monster, teleport away/to, elemi sugár – találatnál és elkerülésnél is);
- aggravate monsters tekercs, vagy aggravate monster gyűrű felhúzása (a szint összes szörnye);
- új szörny születik aggravate gyűrű viselése közben;
- vándorló szörny (azonnal üldöz).

Az ébren lévő szörnyet a Hold monster tekercs állítja meg (ISRUN le, ISHELD be). A `runto` az ISHELD-et is törli.

## 6.5 Célpont
```
find_dest(monster):
  if TYPES[monster.type].carry <= 0 or monster.room == player.room or see_monst(monster):
     return player
  for each item on the level floor (a tárgylista sorrendjében):
     if item is scare monster scroll: continue
     if room(item.pos) == monster.room and rnd(100) < monster.carry:  // tárgyanként külön dobás
        if no other monster has dest == item: return item
  return player
```
- *Forrásbeli hiba:* a „más szörny célja-e” ellenőrzés felülírja a vizsgált szörnyet. Ha egy tárgy foglaltnak bizonyul, a további tárgyakat már a foglaló szörny szobájához hasonlítja (a `carry` esély az eredeti szörnyé marad).
- `dest`-et a `runto` ([6.4](#64-ébresztés)) és a tárgyfelvétel állítja be; szobaváltáskor nem számolódik újra.
- Ha a játékos felveszi a célzott tárgyat, a szörny `dest`-je a játékos lesz ([8.9](08_targyak.md#89-csomag)).
- Kapzsi szörny: ha a szobájában már nincs arany (`gold_val == 0`), a `dest` minden lépésben a játékos lesz.

## 6.6 Mozgás
```
runners():  // az AFTER fázisban, egyszer körönként
  for each monster (a szörnylista sorrendjében):
    if monster.ISHELD or !monster.ISRUN: continue
    old_pos = monster.pos
    if move_monst(monster) == MEGHALT: continue  // pl. a játékost támadva eltűnt (nimfa, leprechaun)
    if monster.ISFLY and dist2(player, monster.pos) >= 3: move_monst(monster)
    if monster.ISTARGET volt és monster.pos != old_pos: ISTARGET le, 'f'-harc vége

move_monst(monster):
  if !monster.ISSLOW or monster.turn: do_chase(monster)
  if monster.ISHASTE: do_chase(monster)
  monster.turn = !monster.turn

do_chase(monster):
  if monster.ISGREED and monster.room.gold_val == 0: monster.dest = player
  target_room = (monster.dest == player) ? player.room : room(monster.dest)
  current_room = monster.room; on_door = (MAP[monster.pos] == DOOR)
  loop:
    if current_room != target_room:                               // más szobában van a cél: a legjobb kijárat felé
       goal = current_room.exits közül a dest-hez (dist2) legközelebbi
       if on_door: current_room = monster.pos folyosórendszere; on_door = false; ismét a ciklus elejére
    else:
       goal = monster.dest
       if monster.type == 'D' and !ISCANC and egy vonalban (vízszintes/függőleges/átló) a játékossal
          and dist2(monster, player) <= 36 and rnd(5) == 0:
          fire_bolt(monster.pos, irány a játékos felé, "flame")   // 7.6
          futás, ismétlés megszakad; regen-számláló = 0; to_death vége, ha nem ez a célpont
          return
    break
  (next_pos, arrived) = chase(monster, goal)
  if arrived:                                                     // chase nem talált jobb mezőt
     if goal == player.pos: return attack(monster)                // 7.1, 6.7
     if goal == monster.dest:
        if monster.dest egy padlón fekvő tárgy:
           felveszi a csomagjába, a mező FLOOR/PASSAGE lesz; monster.dest = find_dest(monster)
        if monster.type != 'F': stop = true
  else if monster.type == 'F': return                             // a Venus flytrap soha nem mozdul el
  relocate(monster, next_pos)
  if stop and monster.pos == monster.dest: monster.ISRUN = false  // pl. a kapzsi szörny ráült az aranyra

chase(monster, goal):  // → (best, megérkezett?); best: ahová lépni fog
  if (monster.ISHUH and rnd(5) != 0) or (monster.type == 'P' and rnd(5) == 0) or (monster.type == 'B' and rnd(2) == 0):
     best = rndmove(monster)                                             // lent
     if rnd(20) == 0: monster.ISHUH = false
     best_dist = dist2(best, goal)
  else:
     best = monster.pos; best_dist = dist2(monster.pos, goal); ties = 1  // ties: hány egyformán jó mezőt láttunk
     for each cell in monster 3×3 környezete (a 0. és 23. sor kivételével), x szerint, azon belül y szerint:
        if !diag_ok(monster.pos, cell): continue
        if !step_ok(MAP/SCREEN jel cell-en): continue                    // fal, kő, szörny nem jó
        if cell-en scare monster tekercs: continue
        if cell-en Xeroc áll: continue
        dist = dist2(cell, goal)
        if dist < best_dist:  best = cell; best_dist = dist; ties = 1
        else if dist == best_dist and rnd(++ties) == 0: best = cell      // egyenletes választás az egyenlők közül
  return (best, best_dist == 0 or best == player.pos)
```
```
rndmove(who):  // zavarodott mozgás (szörny és játékos is)
  target.y = who.pos.y + rnd(3) - 1  // előbb az y, aztán az x!
  target.x = who.pos.x + rnd(3) - 1
  if target == who.pos: return target
  if !diag_ok(who.pos, target) or !step_ok(target) or target-en scare monster tekercs: return who.pos
  return target
```
- A szörny akkor támad, ha a legjobb lépés a játékos mezőjére vinne – vagyis ha szomszédos vele (átlósan csak `diag_ok` szerint).
- Ha nincs közelebbi vagy azonos távolságú mező, helyben marad (`best` = saját mező).
- A szörnyek nem lépnek egymásra (a szörny jele nem `step_ok`), tárgyra igen.

**relocate(monster, new_pos):** ha new_pos ≠ monster.pos: a régi mező helyére `monster.oldch` kerül, `monster.room = room(new_pos)`, majd `monster.oldch` = new_pos képernyőjele, a következő módosításokkal (vakon egyik sem érvényes):
- sötét szobában, ha a régi VAGY az új `oldch` FLOOR, akkor ' ' (így egy tárgy emléke is törlődhet);
- különben a 3×3-as környezeten belül, seefloor mellett a MAP jele.

Ezután a szörny new_pos-ra kerül. A jele kirajzolódik, ha látható (`disguise`), SEEMONST alatt inverzen a típusa.

## 6.7 Speciális támadások (sikeres találat után, ha nincs ISCANC)
```
A aquator: (dmg 0x0) rust_armor(cur_armor)
           rust_armor: if !armor or leather or arm >= 9: nincs hatás
                       else if ISPROT or maintain armor gyűrű: "the rust vanishes instantly"
                       else: arm++; "your armor appears to be weaker now. Oh my!"
F flytrap: találatkor:  player.ISHELD = true; vf_hit++; hpt -= 1
           mellé ütéskor: hpt -= vf_hit                        // fogva tartás alatt a "mellé" is sebez
           vf_hit egyetlen globális számláló; nullázódik a flytrap halálakor és teleportkor
           a flytrap TÍPUS sebzés-stringje is "<vf_hit>x1" lesz (a halála/teleport után "000x0"), így a később
           születő flytrapok ennyi kockával sebeznek (első találatig "%%%x0" = 0 kocka)
I ice:     (dmg 0x0) ISRUN le; ha épp nem dermedt: "you are frozen by the " + név(monster) (7.7); no_command += rnd(2)+2
           if no_command > 50: death("hypothermia")
L leprech: gold -= rnd(50+10*level)+2
           if !save_magic(): gold -= 4 × (rnd(50+10*level)+2)  // négy külön dobás
           gold = max(gold, 0); "your purse feels lighter" (ha változott)
           a leprechaun eltűnik a szintről, a nála lévő tárgyak elvesznek
M medusa:  a pillantás nem támadás, hanem a wake_monster része (6.4); a közelharci támadásai normálisan sebeznek
N nymph:   (dmg 0x0) tárgy = egyenletesen egy mágikus tárgy (8.8) a csomagban, ami nincs használatban
           – az Amulett is mágikus, tehát ellophatja!
           if van: elvesz egy darabot (kötegből egyet), a nimfa eltűnik vele; "she stole <tárgy>!"
           else: nincs hatás, a nimfa marad
R rattle:  if !save_poison():
             if !sustain strength gyűrű: chg_str(-1); "you feel a bite in your leg and now feel weaker"
             else if !to_death: "a bite momentarily weakens you"
V vampire: if rnd(100) < 30: drained = ROLL(1,3); hpt -= drained; maxhp -= drained
             if hpt <= 0: hpt = 1;  if maxhp <= 0: death; "you suddenly feel weaker"
W wraith:  if rnd(100) < 15:
             if exp == 0: death
             lvl--; if lvl == 0: lvl = 1, exp = 0 else exp = xp_next[lvl-1] + 1
             drained = ROLL(1,10); hpt -= drained; maxhp -= drained; if hpt <= 0: hpt = 1; if maxhp <= 0: death
             "you suddenly feel weaker"
           Megjegyzés: az exp így a lvl+1-hez tartozó küszöb fölé kerül (xp_next 0-indexű, 5.2),
           ezért a következő XP-szerzéskor a szint visszaáll (újabb ROLL(1,10) HP-val) – a szintszívás átmeneti
X xeroc:   amíg álcázott: a játékos rálépési kísérlete "wait! That's a xeroc!" üzenettel
           leleplezi (disguise = 'X', ISRUN), a közelharci lépés véget ér; a következő körtől harcol
D dragon:  lásd tűzokádás (7.6)
```

## 6.8 Szörny halála
```
kill_monster(monster):
  exp += monster.exp
  if F: player.ISHELD = false; vf_hit = 0
  if L and fallpos(monster.pos) talál helyet and level >= max_level:
     gold_amount = rnd(50+10*level)+2; if save_magic(): gold_amount += 4 × (rnd(50+10*level)+2)
     az arany a szörny csomagjába kerül (és vele együtt leesik)
  monster.pack minden tárgya: fall(item) – a szörny 3×3 környezetének egy véletlen üres
     FLOOR/PASSAGE mezőjére (a játékosé kivételével); ha nincs hely, a tárgy elvész
  eltávolítás, a mező helyére monster.oldch
  "you have defeated the <name>" (ha látható; ha előtte "hit" üzenet volt: ".  Defeated the <name>")
  szintlépés-ellenőrzés (5.2)
```
