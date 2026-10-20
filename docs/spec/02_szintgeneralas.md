# 2. Szintgenerálás

Sorrend:
```
new_level():
  player.ISHELD = false
  if level > max_level: max_level = level
  clear MAP, SCREEN, monsters (és csomagjaik), floor items
  do_rooms()                    // 2.2–2.5, 2.7: szobák, és szobánként arany + kezdő szörny
  do_passages()                 // 2.6: összekötés és folyosók
  no_food++
  put_things()                  // 2.8: kincses szoba, 9 próba, amulett
  place_traps()                 // 2.9
  place_stairs()                // 2.10; seenstairs = false
  minden szörny: monster.room = room(monster.pos)
  place_player()                // 2.11 + enter_room
  if SEEMONST: turn_see(false)  // a szörnyek az új szinten is azonnal kirajzolódnak
  if ISHALU: visuals()          // a hallucinált jelek azonnal
```

## 2.1 Szektorok
9 szektor 3×3 rácsban. Az `i`-edik szektor (`i = row*3 + col`, 0..8):
- `sector_size = (NUMCOLS/3, NUMLINES/3) = (26, 8)` – egy szektor mérete;
- `sector_corner = ((i%3)*26 + 1, (i/3)*8)` – a szektor bal felső sarka ([1.3](01_adatmodell_es_flagek.md#13-térkép-és-képernyő)).

Minden szektorba pontosan egy szoba kerül; a szoba indexe a szektor indexe.

## 2.2 Szobaflagek
```
for i in 0..8: rooms[i].flags = 0
repeat rnd(4) times: rooms[rnd_room()].flags |= PHANTOM  // 0..3 fantom szoba (ugyanaz a szoba többször is kijöhet)
for each room i (sorrendben):
  if PHANTOM: elhelyezés (2.3), continue
  if rnd(10) < level - 1:
     rooms[i].flags |= DARK
     if rnd(15) == 0: rooms[i].flags = MAZE              // értékadás: a DARK törlődik
  méret (2.3), rajzolás (2.5), arany + kezdő szörny (2.7)

rnd_room():  // véletlen, nem fantom szoba indexe
  repeat candidate = rnd(9) until !PHANTOM(candidate)
  return candidate
```
A labirintus mezői folyosó-mezők (`F_PASS`), ezért a látótér szempontjából folyosóként viselkedik (3×3 látás); a DARK-nak itt nincs jelentősége.

## 2.3 Szobaméret és -hely
A szoba `room.pos` (bal felső sarok, fallal együtt) és `room.size` (méret, fallal együtt) mezője a szektorán belül ([2.1](#21-szektorok)):
```
PHANTOM:  // a fantom szoba nem igazi szoba, csak egyetlen folyosó-csomópont
  repeat:
    room.pos.x = sector_corner.x + rnd(sector_size.x - 2) + 1  // véletlen pont a szektor belsejében,
    room.pos.y = sector_corner.y + rnd(sector_size.y - 2) + 1  //   a szektor szélétől legalább 1 mezőre
  until 0 < room.pos.y < NUMLINES - 1                          // nem az üzenetsorban, nem a státuszsorban
  room.size = (-NUMCOLS, -NUMLINES)                            // "nincs mérete"

MAZE:  // a labirintus kitölti a szektort
  room.size = (sector_size.x - 1, sector_size.y - 1)
  room.pos  = sector_corner
  if room.pos.x == 1: room.pos.x = 0
  if room.pos.y == 0: room.pos.y = 1; room.size.y -= 1  // a 0. sor az üzenetsor

normál:
  repeat:
    room.size.x = rnd(sector_size.x - 4) + 4                          // falakkal együtt, min. 4×4
    room.size.y = rnd(sector_size.y - 4) + 4
    room.pos.x  = sector_corner.x + rnd(sector_size.x - room.size.x)  // úgy, hogy beférjen a szektorba
    room.pos.y  = sector_corner.y + rnd(sector_size.y - room.size.y)
  until room.pos.y != 0                                               // a 0. sor az üzenetsor
```

## 2.4 Összekötés
Szomszédság: csak ortogonális szomszéd szektorok (`adjacent` tábla: 0–1, 0–3, 1–2, 1–4, 2–5, 3–4, 3–6, 4–5, 4–7, 5–8, 6–7, 7–8). A fantom szobák is részt vesznek (folyosó-csomópontként).

Az összekötés egy véletlen feszítőfa a szobák szomszédsági gráfján, utána néhány extra éllel:
```
do_passages():
  connected[a][b] = false minden szobapárra                // a és b közt van-e már folyosó
  in_tree[r]      = false minden szobára                   // r benne van-e már a fában
  from_room = rnd(9); in_tree[from_room] = true; rooms_in_tree = 1
  while rooms_in_tree < 9:
    // from_room egy véletlen, még fán kívüli szomszédja (egyenletesen a jelöltek közül)
    candidates = 0
    for neighbor in 0..8:
      if adjacent[from_room][neighbor] and !in_tree[neighbor] and rnd(++candidates) == 0:
        to_room = neighbor
    if candidates == 0:
       repeat from_room = rnd(9) until in_tree[from_room]  // nincs ilyen szomszéd: új kiinduló szoba a fából
    else:
       in_tree[to_room] = true
       corridor(from_room, to_room)
       connected[from_room][to_room] = connected[to_room][from_room] = true
       rooms_in_tree++                                     // from_room marad a kiinduló szoba!

  // extra élek: néhány további folyosó még össze nem kötött szomszédok közt
  repeat rnd(5) times:
    from_room = rnd(9)
    candidates = 0
    for neighbor in 0..8:
      if adjacent[from_room][neighbor] and !connected[from_room][neighbor] and rnd(++candidates) == 0:
        to_room = neighbor
    if candidates != 0:
       corridor(from_room, to_room)
       connected[from_room][to_room] = connected[to_room][from_room] = true

  passnum()  // az összefüggő folyosóhálózatok megszámozása (F_PNUM), kijáratlistával
```
A kiinduló szoba addig marad ugyanaz, amíg van fán kívüli szomszédja; csak utána választ a fából újat.

A `rnd(++candidates) == 0` minta egyenletesen választ a jelöltek közül anélkül, hogy előre megszámolná őket: az első jelöltet 1/1, a másodikat 1/2, a harmadikat 1/3 eséllyel veszi át, így a végén mindegyik ugyanakkora eséllyel marad. A `rnd` minden jelöltnél lefut.

Átlós lépés ajtóra vagy ajtóról külön tiltás nélkül is lehetetlen: az ajtó két oldalán fal áll, és az átlós lépéshez mindkét ortogonális szomszédnak járhatónak kell lennie.

## 2.5 Szobák rajzolása
```
PHANTOM: nincs rajzolás (a folyosó a room.pos mezőn halad át)
MAZE:    do_maze(room)
normál:  felső és alsó sor (sarkokkal): WALL_H '-'; bal és jobb oszlop (sarkok nélkül): WALL_V '|'
         belső: FLOOR
```
**Labirintus:** rekurzív visszalépéses (DFS) labirintus, 2 mezős lépésekkel; csak folyosó-mezőkből áll, falkarakter nincs. A cellák koordinátái a labirintus bal felső sarkához (`room.pos`) képest értendők, 0..`room.size`.
```
do_maze(room):
  start.y = (rnd(room.size.y) / 2) * 2  // páros kezdőcella; előbb az y, aztán az x
  start.x = (rnd(room.size.x) / 2) * 2
  putpass(room.pos + start)
  dig(start)

dig(cell):  // cell: relatív koordináta a labirintusban
  loop:
    options = 0                                        // ennyi ásható szomszéd van
    for step in [(+2, 0), (-2, 0), (0, +2), (0, -2)]:  // (dy, dx), ebben a sorrendben
      next = cell + step
      if next a labirintuson kívül van (next.y < 0, next.y > room.size.y, next.x < 0, next.x > room.size.x): continue
      if a (room.pos + next) mező már folyosó (F_PASS): continue
      options++
      if rnd(options) == 0: chosen = next              // egyenletes választás az ásható szomszédok közül
    if options == 0: return                            // zsákutca: vissza az előző cellához
    putpass(room.pos + a cell és a chosen közti mező)
    putpass(room.pos + chosen)
    dig(chosen)
```
A labirintus is `putpass`-szal készül, így a mezői is lehetnek rejtettek ([2.6](#26-folyosók)).

## 2.6 Folyosók
Két szomszédos szoba (`room_a`, `room_b`) közti folyosó. A kisebb indexű szobától indul; ha a másik a jobb oldali szomszédja, jobbra halad, ha az alatta lévő, lefelé.
```
corridor(room_a, room_b):
  first = min(room_a, room_b)
  direction = (max(room_a, room_b) == first + 1) ? RIGHT : DOWN
  from = rooms[first]
  to   = rooms[direction == RIGHT ? first + 1 : first + 3]

  if direction == DOWN:
     start = from.pos; end = to.pos                  // fantom szobánál maga a csomópont
     if !PHANTOM(from):                              // ajtó a kiinduló szoba alsó falán
        repeat:
          start.x = from.pos.x + rnd(from.size.x - 2) + 1
          start.y = from.pos.y + from.size.y - 1
        until !MAZE(from) or start folyosó-mező
     if !PHANTOM(to):                                // ajtó a célszoba felső falán
        repeat:
          end.x = to.pos.x + rnd(to.size.x - 2) + 1  // end.y = to.pos.y
        until !MAZE(to) or end folyosó-mező
     step = (0, 1)                                   // a fő irány: lefelé
     length = |start.y - end.y| - 1                  // ennyi mező a fő irányban
     turn_step = (start.x < end.x ? +1 : -1, 0)      // a kanyar iránya: jobbra vagy balra
     turn_length = |start.x - end.x|                 // ennyi mező a kanyarban
  else (direction == RIGHT): ugyanez a tengelyek felcserélésével
     (start a kiinduló szoba jobb oldalfalán, end a célszoba bal oldalfalán, step = (1, 0))
  turn_at = rnd(length - 1) + 1                      // hol fordul a folyosó (a hátralévő hossz szerint)

  PHANTOM(from) ? putpass(start) : door(from, start)
  PHANTOM(to)   ? putpass(end)   : door(to, end)
  pos = start
  while length > 0:
    pos += step
    if length == turn_at:
       repeat turn_length: putpass(pos); pos += turn_step
    putpass(pos); length--
  pos += step  // most pos == end kell legyen

door(room, pos):
  room.exits += pos
  if MAZE(room): return                                               // labirintusnál nincs ajtó
  if rnd(10) + 1 < level and rnd(5) == 0:
     MAP[pos] = (pos a felső vagy alsó falon) ? '-' : '|'; F_REAL le  // titkos ajtó: falnak látszik
  else: MAP[pos] = DOOR

putpass(pos):
  F_PASS be
  if rnd(10) + 1 < level and rnd(40) == 0: F_REAL le  // titkos folyosó-mező (üresnek látszik)
  else: MAP[pos] = PASSAGE
```

## 2.7 Arany és kezdő szörnyek (szobánként, nem fantom szoba)
```
for each room (not PHANTOM):
  has_gold = false
  if rnd(2) == 0 and (not has_amulet or level >= max_level):  // a rnd(2) mindig lefut
     room.gold_val = rnd(50 + 10*level) + 2
     room.gold_pos = find_floor(room, item)                   // üres padló (labirintusnál üres járat)
     place GOLD on room.gold_pos
     has_gold = true
  if rnd(100) < (has_gold ? 80 : 25):
     monster = new_monster(level, initial_list=true)
     place monster on find_floor(room, monster)               // bármely járható mező szörny nélkül – lehet aranyon is
     monster.flags = típus flagjei (nincs ISRUN → alszik)
     give_item(monster)  (lásd 6.3)

find_floor(room /*null = véletlen nem fantom szoba minden próbánál*/, mode):
  loop:
    if room param == null: room = rooms[rnd_room()]
    pos.x = room.pos.x + rnd(room.size.x - 2) + 1  // véletlen mező a falakon belül; előbb az x, aztán az y
    pos.y = room.pos.y + rnd(room.size.y - 2) + 1
    mode == item:    return pos, ha MAP[pos] == (MAZE(room) ? PASSAGE : FLOOR)  (nincs rajta semmi)
    mode == monster: return pos, ha járható és nincs rajta szörny (tárgy, lépcső, rejtett csapda lehet)
```
A kezdő szörny alszik, mert `new_monster` csak a típus flagjeit adja (ISRUN nélkül).

## 2.8 Tárgyak (`put_things`)
```
put_things():
  if has_amulet and level < max_level: return  // csak a legmélyebb elért szinten
  if rnd(20) == 0: treasure_room()             // 5% kincses szoba
  repeat 9:
    if rnd(100) < 36:
      item = new_item()                        // 8.1
      place item on find_floor(null, item)     // labirintus is lehet
  if level >= AMULET_LEVEL and not has_amulet:
      place AMULET on find_floor(null, item)
```

**Kincses szoba:**
```
treasure_room():
  room = random non-phantom room (labirintus is lehet)
  inner_area = (room.size.x - 2) * (room.size.y - 2)  // belső terület, falak nélkül
  MIN = 2; MAX = 10
  extra = min(inner_area - MIN, MAX - MIN)            // a MIN feletti véletlen többlet felső határa; rnd(≤0) = 0 → ilyenkor is MIN tárgy
  item_count = rnd(extra) + MIN
  repeat item_count:
     pos = find_floor(room, item) max. 20 próbával    // sikertelenségnél az előző pos marad (két tárgy egy mezőre kerülhet)
     new_item() a pos-ra
  monster_count = max(rnd(extra) + MIN, item_count + 2)
  monster_count = min(monster_count, inner_area)
  level++                                             // a szörnyek ÉS a tárgyaik is a következő szintről
  repeat monster_count:
     pos = find_floor(room, monster) max. 10 próbával; ha nincs: kihagy
     monster = new_monster(level, initial_list=true); monster.flags |= ISMEAN
     give_item(monster)
  level--
```
Szörnyből mindig legalább kettővel több van, mint tárgyból.

## 2.9 Csapdák
```
if rnd(10) < level:
  trap_count = min(rnd(level/4) + 1, MAX_TRAPS)                  // MAX_TRAPS = 10
  repeat trap_count:
    repeat pos = find_floor(null, item) until MAP[pos] == FLOOR  // labirintusba nem kerül
    F_REAL le                                                    // rejtett: padlóként látszik
    trap_type[pos] = rnd(8)                                      // TRAPDOOR, ARROW, SLEEP, BEAR, TELEPORT, DART, RUST, MYSTERY (F_TMASK)
```

## 2.10 Lépcső
```
pos = find_floor(null, item)  // bármely nem fantom szoba, labirintus is (ott járat-mező)
MAP[pos] = STAIRS
```
A `find_floor` csak a karaktert nézi, így a lépcső egy rejtett csapda mezőjére is kerülhet (annak karaktere FLOOR). Ilyenkor a mező flagjei (csapdatípus, F_REAL nélkül) megmaradnak.

## 2.11 Játékos elhelyezése
```
pos = find_floor(null, monster)  // bármely nem fantom szoba (labirintus is), szörny nélküli járható mező
player.pos = pos; player.ISHELD = false
enter_room(pos)                  // megvilágítás + ébresztés (6.4)
```
A játékos tárgyon, aranyon, lépcsőn vagy rejtett csapdán is kezdhet (a csapda csak rálépéskor sül el, a kezdőmezőn nem).
