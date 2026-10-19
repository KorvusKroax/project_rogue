# 2. Szintgenerálás

Sorrend:
```
new_level():
  player.ISHELD = false
  if level > max_level: max_level = level
  clear MAP, SCREEN, monsters (és csomagjaik), floor items
  do_rooms()                  // 2.2–2.5, 2.7: szobák, és szobánként arany + kezdő szörny
  do_passages()               // 2.6: összekötés és folyosók
  no_food++
  put_things()                // 2.8: kincses szoba, 9 próba, amulett
  place_traps()               // 2.9
  place_stairs()              // 2.10; seenstairs = false
  minden szörny: m.room = room(m.pos)
  place_player()              // 2.11 + enter_room
  if SEEMONST: turn_see(false)  // a szörnyek az új szinten is azonnal kirajzolódnak
  if ISHALU: visuals()          // a hallucinált jelek azonnal
```

## 2.1 Szektorok
9 szektor 3×3 rácsban, index `i = row*3 + col`, méret `bsze = (NUMCOLS/3, NUMLINES/3) = (26, 8)`, bal felső sarok `top = ((i%3)*26 + 1, (i/3)*8)` ([1.3](01_adatmodell_es_flagek.md#13-térkép-és-képernyő)).

## 2.2 Szobaflagek
```
for i in 0..8: flags[i] = 0
repeat rnd(4) times: flags[rnd_room()] |= PHANTOM   // 0..3 fantom szoba (ugyanaz a szoba többször is kijöhet)
for each room i (sorrendben):
  if PHANTOM: elhelyezés (2.3), continue
  if rnd(10) < level - 1:
     flags[i] |= DARK
     if rnd(15) == 0: flags[i] = MAZE      // értékadás: a DARK törlődik
  méret (2.3), rajzolás (2.5), arany + kezdő szörny (2.7)

rnd_room(): repeat r = rnd(9) until !PHANTOM(r); return r
```
A labirintus mezői folyosó-mezők (`F_PASS`), ezért a látótér szempontjából folyosóként viselkedik (3×3 látás); a DARK-nak itt nincs jelentősége.

## 2.3 Szobaméret és -hely
```
PHANTOM:
  repeat:
    pos = (top.x + rnd(bsze.x - 2) + 1, top.y + rnd(bsze.y - 2) + 1)
  until 0 < pos.y < NUMLINES - 1
  size = (-NUMCOLS, -NUMLINES)            // "nincs mérete"

MAZE:
  size = (bsze.x - 1, bsze.y - 1)
  pos  = top;  if pos.x == 1: pos.x = 0
               if pos.y == 0: pos.y = 1; size.y -= 1

normál:
  repeat:
    size = (rnd(bsze.x - 4) + 4, rnd(bsze.y - 4) + 4)       // falakkal együtt, min. 4×4
    pos  = (top.x + rnd(bsze.x - size.x), top.y + rnd(bsze.y - size.y))
  until pos.y != 0                                           // a 0. sor az üzenetsor
```

## 2.4 Összekötés
Szomszédság: csak ortogonális szomszéd szektorok (`conn` tábla: 0–1, 0–3, 1–2, 1–4, 2–5, 3–4, 3–6, 4–5, 4–7, 5–8, 6–7, 7–8). A fantom szobák is részt vesznek (folyosó-csomópontként).
```
do_passages():
  isconn = all false; ingraph = all false
  r1 = rnd(9); ingraph[r1] = true; roomcount = 1
  while roomcount < 9:
    // r1 egy véletlen, még gráfon kívüli szomszédja (egyenletesen)
    j = 0
    for i in 0..8: if conn[r1][i] and !ingraph[i] and rnd(++j) == 0: r2 = i
    if j == 0:
       repeat r1 = rnd(9) until ingraph[r1]   // új kiinduló szoba a gráfból
    else:
       ingraph[r2] = true; corridor(r1, r2); isconn[r1][r2] = isconn[r2][r1] = true
       roomcount++                              // r1 marad a kiinduló szoba!

  // extra élek
  repeat rnd(5) times:
    r1 = rnd(9)
    j = 0
    for i in 0..8: if conn[r1][i] and !isconn[r1][i] and rnd(++j) == 0: r2 = i
    if j != 0: corridor(r1, r2); isconn[r1][r2] = isconn[r2][r1] = true

  passnum()   // az összefüggő folyosóhálózatok megszámozása (F_PNUM), kijáratlistával
```
A kiinduló szoba addig marad ugyanaz, amíg van gráfon kívüli szomszédja; csak utána választ a gráfból újat.

Átlós lépés ajtóra vagy ajtóról külön tiltás nélkül is lehetetlen: az ajtó két oldalán fal áll, és az átlós lépéshez mindkét ortogonális szomszédnak járhatónak kell lennie.

## 2.5 Szobák rajzolása
```
PHANTOM: nincs rajzolás (a folyosó a pos mezőn halad át)
MAZE:    do_maze(room)
normál:  felső és alsó sor (sarkokkal): WALL_H '-'; bal és jobb oszlop (sarkok nélkül): WALL_V '|'
         belső: FLOOR
```
**Labirintus:** rekurzív visszalépéses (DFS) labirintus, 2 mezős lépésekkel; csak folyosó-mezőkből áll, falkarakter nincs.
```
do_maze(room):                      // (Sx,Sy) = room.pos, (Mx,My) = room.size
  sy = (rnd(My) / 2) * 2;  sx = (rnd(Mx) / 2) * 2     // páros kezdőpont
  putpass(Sx+sx, Sy+sy)
  dig(sy, sx)

dig(y, x):
  loop:
    cnt = 0
    for (dy,dx) in [(2,0), (-2,0), (0,2), (0,-2)]:
      ny = y+dy; nx = x+dx
      if ny < 0 or ny > My or nx < 0 or nx > Mx: continue
      if a (Sy+ny, Sx+nx) mező már folyosó (F_PASS): continue
      cnt++; if rnd(cnt) == 0: (ty,tx) = (ny,nx)     // egyenletes választás a jók közül
    if cnt == 0: return
    putpass(a (y,x) és (ty,tx) közti mező)
    putpass(Sy+ty, Sx+tx)
    dig(ty, tx)
```
A labirintus is `putpass`-szal készül, így a mezői is lehetnek rejtettek ([2.6](#26-folyosók)).

## 2.6 Folyosók
```
corridor(r1, r2):                       // rm = a kisebb indexű; 'r' ha r2 == rm+1, különben 'd'
  rpf = rooms[rm]; rpt = rooms[rm+1 vagy rm+3]
  if 'd' (lefelé):
     spos = rpf.pos; epos = rpt.pos                       // fantom szobánál maga a pont
     if !PHANTOM(rpf): repeat spos = (rpf.pos.x + rnd(rpf.w - 2) + 1, rpf.pos.y + rpf.h - 1)
                       until !MAZE(rpf) or spos folyosó-mező
     if !PHANTOM(rpt): repeat epos.x = rpt.pos.x + rnd(rpt.w - 2) + 1     // epos.y = rpt.pos.y
                       until !MAZE(rpt) or epos folyosó-mező
     del = (0, 1); distance = |spos.y - epos.y| - 1
     turn_delta = (spos.x < epos.x ? +1 : -1, 0); turn_distance = |spos.x - epos.x|
  else ('r', jobbra): ugyanez a tengelyek felcserélésével
     (spos a jobb oldalfalon, epos a bal oldalfalon, del = (1, 0))
  turn_spot = rnd(distance - 1) + 1                        // hol fordul a folyosó

  PHANTOM(rpf) ? putpass(spos) : door(rpf, spos)
  PHANTOM(rpt) ? putpass(epos) : door(rpt, epos)
  curr = spos
  while distance > 0:
    curr += del
    if distance == turn_spot:
       repeat turn_distance: putpass(curr); curr += turn_delta
    putpass(curr); distance--
  curr += del                                              // most curr == epos kell legyen

door(room, P):
  room.exits += P
  if MAZE(room): return                                    // labirintusnál nincs ajtó
  if rnd(10) + 1 < level and rnd(5) == 0:
     MAP[P] = (P a felső vagy alsó falon) ? '-' : '|'; F_REAL le    // titkos ajtó
  else: MAP[P] = DOOR

putpass(P):
  F_PASS be
  if rnd(10) + 1 < level and rnd(40) == 0: F_REAL le      // titkos folyosó-mező (üresnek látszik)
  else: MAP[P] = PASSAGE
```

## 2.7 Arany és kezdő szörnyek (szobánként, nem fantom szoba)
```
for each room (not PHANTOM):
  has_gold = false
  if rnd(2) == 0 and (not has_amulet or level >= max_level):   // a rnd(2) mindig lefut
     gold = rnd(50 + 10*level) + 2
     place GOLD on find_floor(room, item)              // üres padló (labirintusnál üres járat)
     has_gold = true
  if rnd(100) < (has_gold ? 80 : 25):
     m = new_monster(level, initial_list=true)
     place on find_floor(room, monster)                 // bármely járható mező szörny nélkül – lehet aranyon is
     m.flags = típus flagjei (nincs ISRUN → alszik)
     give_item(m)  (lásd 6.3)

find_floor(room /*null = véletlen nem fantom szoba minden próbánál*/, mode):
  loop:
    P = room.pos + (1 + rnd(room.w - 2), 1 + rnd(room.h - 2))   // a falakon belül
    mode == item:    elfogad, ha MAP[P] == (MAZE(room) ? PASSAGE : FLOOR)  (nincs rajta semmi)
    mode == monster: elfogad, ha járható és nincs rajta szörny (tárgy, lépcső, rejtett csapda lehet)
```
A kezdő szörny alszik, mert `new_monster` csak a típus flagjeit adja (ISRUN nélkül).

## 2.8 Tárgyak (`put_things`)
```
put_things():
  if has_amulet and level < max_level: return      // csak a legmélyebb elért szinten
  if rnd(20) == 0: treasure_room()                 // 5% kincses szoba
  repeat 9:
    if rnd(100) < 36:
      it = new_item()                              // 8.1
      place on find_floor(null, item)              // labirintus is lehet
  if level >= AMULET_LEVEL and not has_amulet:
      place AMULET on find_floor(null, item)
```

**Kincses szoba:**
```
treasure_room():
  room = random non-phantom room (labirintus is lehet)
  area = (width-2)*(height-2)                       // belső terület, falak nélkül
  MIN = 2; MAX = 10
  spots = min(area - MIN, MAX - MIN)                // rnd(≤0) = 0 → ilyenkor is MIN tárgy
  n_items = rnd(spots) + MIN
  repeat n_items:
     P = find_floor(room, item) max. 20 próbával   // sikertelenségnél az előző P marad (két tárgy egy mezőre kerülhet)
     new_item() P-re
  n_mon = max(rnd(spots) + MIN, n_items + 2)
  n_mon = min(n_mon, area)
  level++                                           // a szörnyek ÉS a tárgyaik is a következő szintről
  repeat n_mon:
     P = find_floor(room, monster) max. 10 próbával; ha nincs: kihagy
     m = new_monster(level, initial_list=true); m.flags |= ISMEAN
     give_item(m)
  level--
```
Szörnyből mindig legalább kettővel több van, mint tárgyból.

## 2.9 Csapdák
```
if rnd(10) < level:
  n = min(rnd(level/4) + 1, MAX_TRAPS)      // MAX_TRAPS = 10
  repeat n:
    repeat P = find_floor(null, item) until MAP[P] == FLOOR   // labirintusba nem kerül
    MAP[P] = FLOOR | HIDDEN (+ trap_type)                     // padlóként látszik
    trap_type[P] = rnd(8)      // TRAPDOOR, ARROW, SLEEP, BEAR, TELEPORT, DART, RUST, MYSTERY
```

## 2.10 Lépcső
```
P = find_floor(null, item)          // bármely nem fantom szoba, labirintus is (ott járat-mező)
MAP[P] = STAIRS
```
A `find_floor` csak a karaktert nézi, így a lépcső egy rejtett csapda mezőjére is kerülhet (annak karaktere FLOOR). Ilyenkor a mező flagjei (csapdatípus, F_REAL nélkül) megmaradnak.

## 2.11 Játékos elhelyezése
```
P = find_floor(null, monster)       // bármely nem fantom szoba (labirintus is), szörny nélküli járható mező
player.pos = P; player.ISHELD = false
enter_room(P)                       // megvilágítás + ébresztés (6.4)
```
A játékos tárgyon, aranyon, lépcsőn vagy rejtett csapdán is kezdhet (a csapda csak rálépéskor sül el, a kezdőmezőn nem).
