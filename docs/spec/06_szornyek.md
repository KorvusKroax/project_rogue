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
WANDER  = "KEBSH0ROZ0CQA0Y0TWP0UMVGJ0"     // 0 = nem lehet vándorló

new_monster(lvl_d, list):
  loop:
    i = lvl_d + rnd(10) - 6
    if i < 0:  i = rnd(5)
    if i > 25: i = rnd(5) + 21
    if list[i] != 0: break
  t = TYPES[list[i]]
  mod = max(lvl_d - AMULET_LEVEL, 0)
  m.lvl = t.lvl + mod
  m.maxhp = m.hpt = ROLL(m.lvl, 8)
  m.arm = t.arm - mod
  m.dmg = t.dmg; m.str = t.str; m.flags = t.flags
  m.exp = t.exp + mod*10 + expmod(m.lvl, m.maxhp)
  if lvl_d > 29: m.flags |= ISHASTE
  m.turn = true                                // lassítva az első körben lép
  m.oldch = a mező jelenlegi képernyőjele
  m a szörnylista ELEJÉRE kerül                 // a runners így a legújabbal kezd
  if aggravate ring worn: runto(m)              // ISRUN, ISHELD le, dest = find_dest
  m.disguise = type
  if type == 'X': m.disguise = rnd_thing()
       // egyenletesen: ! ? = / : ) ] % *   és a 26. szinttől a , (amulett) is
       // (POTION, SCROLL, RING, STICK, FOOD, WEAPON, ARMOR, STAIRS, GOLD, [AMULET])

expmod(lvl, hp):
  mod = (lvl == 1) ? hp/8 : hp/6
  if lvl > 9: mod *= 20 else if lvl > 6: mod *= 4
  return mod
```

**Vándorló szörny démon:**
```
// induláskor: időzítő spread(70) (AFTER fázis); lejártakor elindul a rollwand démon (BEFORE fázis)
rollwand():                                   // BEFORE fázis, körönként egyszer
  if ++between >= 4:
     if ROLL(1,6) == 4:
        wanderer()
        rollwand démon le; új időzítő spread(70), ezúttal a BEFORE fázisban
     between = 0

wanderer():
  hely: find_floor(null, monster), ismételve amíg nem a játékos szobája
  m = new_monster(level, WANDER)
  if SEEMONST: m kirajzolása inverzen (hallucinálva véletlen betűvel)
  runto(m)                                    // ISRUN; dest = find_dest(m) – általában a játékos
  nem kap tárgyat
```

## 6.3 Kezdő tárgy
```
give_item(m):
  if level >= max_level and rnd(100) < carry: m.pack += new_item()
```

## 6.4 Ébresztés

**Mikor hívódik a `wake_monster`:**
1. **Szobába belépéskor és szoba elhagyásakor** (`enter_room` / `leave_room` → `door_open`): a szoba **összes** szörnyére, világos és sötét szobában egyaránt (fantom szobánál nem).
2. **Minden játékos-akció előtt** (`look(TRUE)` a parancsciklusban): a játékos körüli **8 szomszédos** mezőn álló, látható szörnyekre. Átlós szomszéd csak akkor, ha nem sarkon át esik.

```
wake_monster(m):
  // 1. aljas szörny
  if !m.ISRUN and rnd(3) != 0 and m.ISMEAN and !m.ISHELD
     and !wearing(STEALTH) and !player.ISLEVIT:
       m.dest = player; m.flags |= ISRUN
  // 2. Medusa pillantása – minden hívásnál próbálkozik, amíg el nem sütötte (ISFOUND)
  if m.type == 'M' and !player.ISBLIND and !player.ISHALU
     and !m.ISFOUND and !m.ISCANC and m.ISRUN:
       if (a JÁTÉKOS szobája világos) or dist2(m, player) < 3:
          m.flags |= ISFOUND                       // a mentődobás előtt → csak egy esély
          if !save_magic():
             ISHUH; ha már zavarodott: az időzítő +spread(20), különben új időzítő spread(20)
             név(m) + "'s gaze has confused you"   // pl. "the medusa's gaze …"; ha név(m) == "it": "its gaze …"
  // 3. kapzsi szörny – nincs 2/3 esély, a stealth és a lebegés sem védi ki, ISHELD-et sem nézi
  if m.ISGREED and !m.ISRUN:
       m.flags |= ISRUN
       m.dest = (a JÁTÉKOS szobájában van arany) ? annak helye : player

runto(m):           // a "kényszerített" ébresztés
  m.flags |= ISRUN; m.flags &= ~ISHELD; m.dest = choose_dest(m)
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
find_dest(m):
  if TYPES[m.type].carry <= 0 or m.room == player.room or see_monst(m):
     return player
  for each obj on the level floor (a tárgylista sorrendjében):
     if obj is scare monster scroll: continue
     if room(obj.pos) == m.room and rnd(100) < carry:      // tárgyanként külön dobás
        if no other monster has dest == obj: return obj
  return player
```
- *Forrásbeli hiba:* a „más szörny célja-e” ellenőrzés felülírja a vizsgált szörnyet. Ha egy tárgy foglaltnak bizonyul, a további tárgyakat már a foglaló szörny szobájához hasonlítja (a `carry` esély az eredeti szörnyé marad).
- `dest`-et a `runto` ([6.4](#64-ébresztés)) és a tárgyfelvétel állítja be; szobaváltáskor nem számolódik újra.
- Ha a játékos felveszi a célzott tárgyat, a szörny `dest`-je a játékos lesz ([8.9](08_targyak.md#89-csomag)).
- Kapzsi szörny: ha a szobájában már nincs arany (`gold_val == 0`), a `dest` minden lépésben a játékos lesz.

## 6.6 Mozgás
```
runners():                                   // az AFTER fázisban, egyszer körönként
  for each m (a szörnylista sorrendjében):
    if m.ISHELD or !m.ISRUN: continue
    orig = m.pos
    if move_monst(m) == MEGHALT: continue     // pl. a játékost támadva eltűnt (nimfa, leprechaun)
    if m.ISFLY and dist2(player, m.pos) >= 3: move_monst(m)
    if m.ISTARGET volt és m.pos != orig: ISTARGET le, 'f'-harc vége

move_monst(m):
  if !m.ISSLOW or m.turn: do_chase(m)
  if m.ISHASTE: do_chase(m)
  m.turn = !m.turn

do_chase(m):
  if m.ISGREED and m.room.gold_val == 0: m.dest = player
  target_room = (m.dest == player) ? player.room : room(m.dest)
  rer = m.room; on_door = (MAP[m.pos] == DOOR)
  loop:
    if rer != target_room:
       goal = rer.exits közül a dest-hez (dist2) legközelebbi
       if on_door: rer = m.pos folyosórendszere; on_door = false; ismét a ciklus elejére
    else:
       goal = m.dest
       if m is 'D' and !ISCANC and egy vonalban (vízszintes/függőleges/átló) a játékossal
          and dist2(m, player) <= 36 and rnd(5) == 0:
          fire_bolt(m.pos, irány a játékos felé, "flame")       // 7.6
          futás, ismétlés megszakad; regen-számláló = 0; to_death vége, ha nem ez a célpont
          return
    break
  (T, arrived) = chase(m, goal)
  if arrived:                                  // chase nem talált jobb mezőt
     if goal == player.pos: return attack(m)    // 7.1, 6.7
     if goal == m.dest:
        if m.dest egy padlón fekvő tárgy:
           felveszi a csomagjába, a mező FLOOR/PASSAGE lesz; m.dest = find_dest(m)
        if m != 'F': stop = true
  else if m is 'F': return                     // a Venus flytrap soha nem mozdul el
  relocate(m, T)
  if stop and m.pos == m.dest: m.ISRUN = false   // pl. a kapzsi szörny ráült az aranyra

chase(m, goal):                                // → (T, megérkezett?)
  if (m.ISHUH and rnd(5) != 0) or (m is 'P' and rnd(5) == 0) or (m is 'B' and rnd(2) == 0):
     T = rndmove(m)                            // lent
     if rnd(20) == 0: m.ISHUH = false
     d = dist2(T, goal)
  else:
     T = m.pos; d = dist2(m.pos, goal); ties = 1
     for each N in m 3×3 környezete (a 0. és 23. sor kivételével), x szerint, azon belül y szerint:
        if !diag_ok(m.pos, N): continue
        if !step_ok(MAP/SCREEN jel N-en): continue        // fal, kő, szörny nem jó
        if N-en scare monster tekercs: continue
        if N-en Xeroc áll: continue
        dn = dist2(N, goal)
        if dn < d:  T = N; d = dn; ties = 1
        else if dn == d and rnd(++ties) == 0: T = N         // egyenletes választás az egyenlők közül
  return (T, d == 0 or T == player.pos)
```
```
rndmove(who):                                // zavarodott mozgás (szörny és játékos is)
  T = who.pos + (rnd(3) - 1, rnd(3) - 1)
  if T == who.pos: return T
  if !diag_ok(who.pos, T) or !step_ok(T) or T-n scare monster tekercs: return who.pos
  return T
```
- A szörny akkor támad, ha a legjobb lépés a játékos mezőjére vinne – vagyis ha szomszédos vele (átlósan csak `diag_ok` szerint).
- Ha nincs közelebbi vagy azonos távolságú mező, helyben marad (`T` = saját mező).
- A szörnyek nem lépnek egymásra (a szörny jele nem `step_ok`), tárgyra igen.

**relocate(m, T):** ha T ≠ m.pos: a régi mező helyére `m.oldch` kerül, `m.room = room(T)`, majd `m.oldch` = T képernyőjele, a következő módosításokkal (vakon egyik sem érvényes):
- sötét szobában, ha a régi VAGY az új `oldch` FLOOR, akkor ' ' (így egy tárgy emléke is törlődhet);
- különben a 3×3-as környezeten belül, seefloor mellett a MAP jele.

Ezután a szörny T-re kerül. A jele kirajzolódik, ha látható (`disguise`), SEEMONST alatt inverzen a típusa.

## 6.7 Speciális támadások (sikeres találat után, ha nincs ISCANC)
```
A aquator: (dmg 0x0) rust_armor(cur_armor)
           rust_armor: if !armor or leather or arm >= 9: nincs hatás
                       else if ISPROT or maintain armor gyűrű: "the rust vanishes instantly"
                       else: arm++; "your armor appears to be weaker now. Oh my!"
F flytrap: találatkor:  player.ISHELD = true; vf_hit++; hpt -= 1
           mellé ütéskor: hpt -= vf_hit         // fogva tartás alatt a "mellé" is sebez
           vf_hit egyetlen globális számláló; nullázódik a flytrap halálakor és teleportkor
           a flytrap TÍPUS sebzés-stringje is "<vf_hit>x1" lesz (a halála/teleport után "000x0"), így a később
           születő flytrapok ennyi kockával sebeznek (első találatig "%%%x0" = 0 kocka)
I ice:     (dmg 0x0) ISRUN le; ha épp nem dermedt: "you are frozen by the " + név(m) (7.7); no_command += rnd(2)+2
           if no_command > 50: death("hypothermia")
L leprech: gold -= rnd(50+10*level)+2
           if !save_magic(): gold -= 4 × (rnd(50+10*level)+2)    // négy külön dobás
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
V vampire: if rnd(100) < 30: d = ROLL(1,3); hpt -= d; maxhp -= d
             if hpt <= 0: hpt = 1;  if maxhp <= 0: death; "you suddenly feel weaker"
W wraith:  if rnd(100) < 15:
             if exp == 0: death
             lvl--; if lvl == 0: lvl = 1, exp = 0 else exp = xp_next[lvl-1] + 1
             d = ROLL(1,10); hpt -= d; maxhp -= d; if hpt <= 0: hpt = 1; if maxhp <= 0: death
             "you suddenly feel weaker"
           Megjegyzés: az exp így a lvl+1-hez tartozó küszöb fölé kerül (xp_next 0-indexű, 5.2),
           ezért a következő XP-szerzéskor a szint visszaáll (újabb ROLL(1,10) HP-val) – a szintszívás átmeneti
X xeroc:   amíg álcázott: a játékos rálépési kísérlete "wait! That's a xeroc!" üzenettel
           leleplezi (disguise = 'X', ISRUN), a közelharci lépés véget ér; a következő körtől harcol
D dragon:  lásd tűzokádás (7.6)
```

## 6.8 Szörny halála
```
kill_monster(m):
  exp += m.exp
  if F: player.ISHELD = false; vf_hit = 0
  if L and fallpos(m.pos) talál helyet and level >= max_level:
     arany = rnd(50+10*level)+2; if save_magic(): += 4 × (rnd(50+10*level)+2)
     az arany a szörny csomagjába kerül (és vele együtt leesik)
  m.pack minden tárgya: fall(obj) – a szörny 3×3 környezetének egy véletlen üres
     FLOOR/PASSAGE mezőjére (a játékosé kivételével); ha nincs hely, a tárgy elvész
  eltávolítás, a mező helyére m.oldch
  "you have defeated the <name>" (ha látható; ha előtte "hit" üzenet volt: ".  Defeated the <name>")
  szintlépés-ellenőrzés (5.2)
```
