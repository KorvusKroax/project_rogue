# 7. Harc

## 7.1 Közelharci találat
```
attack(attacker, defender, weapon /*null = fegyvertelen*/, thrown):
  for each segment in damage_string:
    roll = rnd(20) + hit_bonus
    hit_bonus = weapon ? weapon.hplus : 0
              + str_plus[attacker.str]               // szörnynél 0
              + (player, weapon kézben, !thrown ? sum(dexterity rings.arm) : 0)
    if !defender.ISRUN: roll += 4                    // alvó/fogva tartott szörny – és ISRUN nélküli játékos is (1.2.1)
    required = (20 - attacker.lvl) - defender_armor  // ennyit kell legalább dobni a találathoz
    if roll >= required:
       dmg = ROLL(segment) + (weapon ? weapon.dplus : 0) + add_dam[attacker.str]
             + (player, weapon kézben, !thrown ? sum(increase damage rings.arm) : 0)
       dmg = max(dmg, 0)
       defender.hpt -= dmg
       on_hit effects (CANHUH, speciális támadás)
       if defender.hpt <= 0: kill
```
- A regen-számláló ([5.3](05_jatekos.md#53-hp-regeneráció)) minden harci forduló elején nullázódik, akár a játékos, akár a szörny támad, találattól függetlenül.
- Védő páncél: játékosnál `player_armor()` ([5.5](05_jatekos.md#55-effektív-páncél-védekezéskor)), szörnynél `monster.arm`.
- Játékos támad: a szörny azonnal ISRUN-t kap és ISHELD-je törlődik (ezért a +4 gyakorlatilag nem érvényesül).
- Szörny támad: `weapon = null`, `str = 10` → `str_plus = 0`, `add_dam = 0`.
- Fegyveres közelharc sebzés-stringje: `weapon.damage`; fegyvertelen: `player.dmg` ("1x4"); szörnynél `monster.dmg` szegmensenként.
- CANHUH: sikeres játékos-ütésnél a szörny ISHUH-t kap, CANHUH törlődik, „your hands stop glowing red” / „the <szörny> appears confused”.
- Xeroc leleplezés: közelharcnál csak leleplezi, sebzés nincs, a lépés véget ér.

## 7.2 Fight (`f`) leállása
Lásd [4.2.2](04_jatekciklus_es_parancsok.md#422-harc-és-célzás): célpont halála/eltűnése/elmozdulása, más szörny támadása, vagy (nem kamikaze) `hpt <= max_hit_this_fight` (az eddigi legnagyobb egyszeri sebzés ebben a harcban).

## 7.3 Mentődobások
```
save_magic(who):  rnd(20)+1 >= 17 - who.lvl/2 - (player ? sum(protection rings.arm) : 0)
save_poison():    rnd(20)+1 >= 14 - player.lvl/2
```

## 7.4 Hajítás (`t`)
```
throw():
  dir = ask_direction()                                                  // ESC: nincs idő
  if ISHUH and rnd(5) == 0: dir = random of 8 (nem (0,0))
  chosen = ask_item("which object do you want to throw? (* for list)")   // * lista csak fegyver; bármit elfogad; ESC: nincs idő
  if chosen in use:
     if chosen.ISCURSED: "you can't. It appears to be cursed"; return TIME
     unequip(chosen)                                                     // fegyver le; páncél le (waste_time, 4.2.3); gyűrű le hatással együtt
  missile = (chosen.count > 1) ? split_one(chosen) : remove_from_pack(chosen)  // köteg: csak egy darab repül
  pos = player.pos                                                       // a tárgy aktuális helye repülés közben
  loop:
    next = pos + dir
    if not walkable(next) or next is DOOR or monster at next: break
    pos = next; animate(pos) ha látható és nem terse
  landing = (monster at next) ? next : (next is wall/door ? next : pos)  // a drop_near innen keres helyet
  if monster at next:
     if xeroc álcázott: "wait! That's a xeroc!" (de a dobás lefut)
     hit = throw_attack(missile, monster)
     monster.ISRUN = true; regen megszakad
     if hit: if CANHUH: confuse monster, CANHUH le; missile elfogy; return TIME
  drop_near(missile, landing)
  return TIME

throw_attack(missile, monster):
  if missile is arrow and cur_weapon is its launcher (bow):
       dice = missile.hurldmg (2x3); hit_bonus = missile.hplus + bow.hplus; dmg_bonus = missile.dplus + bow.dplus
  else if missile is arrow: dice = missile.damage (1x1); hit_bonus = missile.hplus; dmg_bonus = missile.dplus
  else if missile is weapon: dice = missile.hurldmg; hit_bonus = missile.hplus; dmg_bonus = missile.dplus
  else: dice = "0x0"; hit_bonus = dmg_bonus = 0
  roll = rnd(20) + hit_bonus + str_plus[str] (+4 ha !monster.ISRUN)
  hit if roll >= (20 - player.lvl) - monster.arm
  dmg = ROLL(dice) + dmg_bonus + add_dam[str]  // gyűrűk (dex/dmg) NEM számítanak

drop_near(item, center):
  free_cells = center 3×3 környezetéből azok a mezők, amelyek MAP-karaktere FLOOR vagy PASSAGE
               (tehát nincs rajta tárgy), a játékos mezője kivételével
  if free_cells: item.pos = free_cells[rnd(len(free_cells))] else "the <név> vanishes as it hits the ground"
  // a <név> a fegyvertáblából jön (weap_info[item.which]), nem fegyver tárgynál is – forrásbeli hiba
```
- A repülés a képernyőjelet nézi: a tárgynak álcázott Xeroc jele járható, ezért a dobott tárgy áthalad rajta. Ugyanez igaz a varázslövedékre és az „első szörnyig” haladó pálcákra ([8.7](08_targyak.md#87-pálcákbotok-z)).

## 7.5 Csapda-támadás (nyíl/dart)
```
trap_hit(eff_lvl): rnd(20) + 1 >= (20 - eff_lvl) - 10
// a védekező páncél a játékos ALAP értéke (pstats.s_arm = 10), a viselt páncél és a
// védelem-gyűrű NEM számít!
arrow: trap_hit(player.lvl - 1) → hpt -= ROLL(1,6) "oh no! An arrow shot you" (halál: "an arrow killed you")
       különben: 1 nyílvessző leesik (fall) "an arrow shoots past you"
dart:  trap_hit(player.lvl + 1) → hpt -= ROLL(1,4); if !sustain and !save_poison(): chg_str(-1)
       "a small dart just hit you in the shoulder"
       különben "a small dart whizzes by your ear and vanishes"
```

## 7.6 Elemi sugár (lightning / fire / cold pálca, sárkánytűz)
```
fire_bolt(origin, dir, name /* "bolt" | "flame" | "ice" */):
  pos = origin; steps = 0                                      // pos: a sugár feje; steps: megtett lépések (max. 6)
  can_hit_hero = (origin != player)                            // a játékos saját sugara először csak szörnyet érhet
  changed = false; used = false                                // used: a sugár már eltalált valakit
  while steps < 6 and !used:
    pos += dir
    if MAP[pos] is WALL/ROCK or (DOOR and pos != player.pos):  // visszapattanás
       if !changed: can_hit_hero = !can_hit_hero
       changed = false
       dir = -dir                                              // teljes megfordulás (mindkét tengely), átlósnál is
       "the <name> bounces"                                    // pos a falon marad, a következő pos += dir visszahozza;
       continue                                                // a pattanás NEM számít a 6 lépésbe
    steps++
    if !can_hit_hero and monster at pos:
       can_hit_hero = true; changed = !changed
       if !save_throw_magic(monster):
          used = true
          if name == "flame" and monster is 'D': "the flame bounces off the dragon" (nincs sebzés)
          else: hit_monster(dobott fegyverként: 6x6 sebzés, +100 találat)  // gyakorlatilag mindig talál
       else:
          if origin == player: runto(monster)
          "the <name> whizzes past <monster>"
    else if can_hit_hero and pos == player.pos:
       can_hit_hero = false; changed = !changed
       if !save_magic(): hpt -= ROLL(6,6); used = true; "you are hit by the <name>"
            halál: death(origin == player ? "bolt" : a kilövő szörny)
       else: "the <name> whizzes by you"
    rajz: '-' '|' '/' '\' az iránytól függően
  a sugár útját visszarajzolja az eredeti mezőkre
```
A sugár egy körben legfeljebb egyszer hat valakire, utána a „cél” felváltva szörny/játékos. A játékos sugara csak visszapattanás után találhatja el a játékost; a sárkányé először a játékost.
- A pattanás a képernyőjel alapján történik: ' ' (kő, rejtett folyosó), `|`, `-` és ajtó.
- A szörnyet érő sugár a **játékos** dobott támadásaként fut le (`fight`), akkor is, ha a sárkány visszapattant lángja talál el egy másik szörnyet. Ezért ilyenkor is a játékos erő-bónusza számít, a CANHUH hatás elsül, és a szörny halálakor a játékos kapja az XP-t. Üzenet: „the <bolt|flame|ice> hits <szörny>”.

## 7.7 Harci üzenetek
```
név(monster):  // set_mname
  if !see_monst(monster) and !SEEMONST: return terse ? "it" : "something"
  if ISHALU: a képernyőn látott (véletlen) betűhöz tartozó szörnynév
  return "the " + monster.name

hit(támadó, célpont):  // támadó/célpont: null = "you"
  if to_death: nincs üzenet  // 'f'-harc közben csend
  verb = terse ? (" hit") : h_names[rnd(4) + (támadó == null ? 0 : 4)]
  üzenet: Támadó(nagybetűvel) + verb + (terse ? "" : célpont)
miss(...): ugyanígy m_names-szel; terse alak: " miss" / " misses"

h_names = [" scored an excellent hit on ", " hit ", " have injured ", " swing and hit ",  // you …
           " scored an excellent hit on ", " hit ", " has injured ", " swings and hits "]  // a szörny …
m_names = [" miss", " swing and miss", " barely miss", " don't hit",  // you …
           " misses", " swings and misses", " barely misses", " doesn't hit"]  // a szörny …
```
Példák: „You swing and hit the orc”, „The orc barely misses you”, terse: „You hit”.

- **Dobott tárgy:** találat: „the <fegyver> hits <szörny>” (nem fegyvernél „you hit <szörny>”); mellé: „the <fegyver> misses <szörny>” / „you missed <szörny>”.
- **Xeroc leleplezése:** „wait!  That's a xeroc!” (hallucinálva: „heavy!  That's a nasty critter!”).
- **Monster confusion:** sikeres ütés után „your hands stop glowing red”, majd „<szörny> appears confused” (vakon nem).
- **Halál:** „you have defeated <szörny>” (terse: „defeated <szörny>”); ha egy „hit” üzenet után jön: „.  Defeated <szörny>”.
- Az Ice monster találata nem ír „hits you”-t, csak „you are frozen” + (nem terse) „ by the ” + név(m) – a név maga is „the”-vel kezdődik, így a szöveg „you are frozen by the the ice monster”.
- A terse módban és nem `f`-harcban az egymást követő találati üzenetek egy sorba fűződnek (`has_hit`).
