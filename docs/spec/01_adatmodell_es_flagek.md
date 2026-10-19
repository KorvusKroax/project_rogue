# 1. Adatmodell és flagek

## 1.1 Rekordok

### 1.1.1 Room (9 db, szektoronként egy)
```
Room {
  pos: (x,y)         // bal felső sarok (fallal együtt); fantom szobánál maga a folyosó-pont
  size: (w,h)        // méret falakkal együtt
  flags:
    PHANTOM           // "gone" szoba (ISGONE): csak egy folyosó-csomópont
    DARK              // a szoba sötét (ISDARK)
    MAZE              // labirintus (ISMAZE)
  exits[]            // ajtók/kijáratok koordinátái (szörny útkereséshez)
  gold_pos, gold_val // arany a szobában (ha van)
}
// összekötéshez (csak generálás közben): conn[9][9] szomszédsági tábla, isconn[9][9], ingraph[9]
// folyosórendszerek: passages[] – minden összefüggő folyosóhálózat külön "szobaként" szerepel
//   (r_flags = ISGONE|ISDARK), ez kell a drain life-hoz és a szörnyek útkereséséhez
```

### 1.1.2 Creature (játékos és szörny közös rekord)
```
Creature {
  str, exp, lvl, arm, dmg(string), hpt, maxhp
  pos, room, flags, pack(list<Item>), oldch   // oldch: mi volt a mezőn alatta
  // csak szörny:
  type ('A'..'Z'), name, carry(%), dest(pos vagy ref), disguise(char), turn(bool)
  // segédmezők:
  // a Venus flytrap számlálója (vf_hit) globális, a GameState-ben van
}
```
Játékos-specifikus állapot a `GameState`-ben (lásd lent).

### 1.1.3 Item
```
Item {
  type      // POTION, SCROLL, FOOD, WEAPON, ARMOR, STICK, RING, GOLD, AMULET
  which     // altípus index a kategórián belül
  pos       // padlón
  packch    // csomagbetű 'a'..
  text      // azonosítatlan név (szín, felirat, kő, anyag)
  launch    // kilövő (nyílnál: BOW)
  damage    // kézben ("NxM")
  hurldmg   // dobva
  count     // darabszám
  hplus, dplus
  arm       // páncélnál: védelem; pálcánál: töltés; aranynál: érték; gyűrűnél: mod
  flags     // ISCURSED, ISKNOW, ISMISL, ISMANY, ISFOUND, ISPROT
  group     // kötegazonosító
  label     // játékos által adott név ('c')
}
```

### 1.1.4 GameState
```
GameState {
  level, max_level, gold, turn
  player: Creature
  cur_weapon, cur_armor, cur_ring[LEFT], cur_ring[RIGHT]
  food_left, hungry_state (0..3), no_food, regen_counter
  has_amulet
  str_max                         // "Str: cur(max)"
  timers: { confused, hallucinating, see_invis, blind, levitating, hasted,
            sleeping/fainted/frozen (no_command), bear_trap, monster_detect }
  wander_timer, wander_check_tick
  know[type][which], guess[type][which]  // azonosítás, 'c' elnevezések
  names[type][which]                     // azonosítatlan nevek (színek, kövek...)
  stick_is_staff[which]                  // bot vagy pálca ebben a játékban
  rooms[9], monsters list, floor items list, traps
  last_command, last_message, count_prefix
  can_huh (CANHUH), max_hit_this_fight
}
```

---

## 1.2 Flagek

### 1.2.1 Játékos
| flag | jelentés | be | ki |
|---|---|---|---|
| ISRUN | „ébren van” – egyetlen hatása: ha hiányzik, a szörnyek +4-gyel találják el (a cselekvőképességet a `no_command` számláló kezeli) | alap; ájulás/alvás/fagyás lejártakor | alvás (tekercs, gáz), fagyás, gyorsítás-ájulás, éhség-állapotváltás |
| ISHUH | zavarodott: mozgásnál 80% véletlen irány (`dx,dy = rnd(3)-1`; (0,0) nem visz időt), dobásnál/`^`-nál 20% véletlen irány | bájital, Medusa | időzítő; időtartamok összeadódnak |
| CANHUH | a következő sikeres ütés összezavarja a szörnyet | Monster confusion tekercs | első sikeres ütés (közel vagy dobás) |
| CANSEE | látja a láthatatlant | bájital, gyűrű | időzítő / gyűrű le |
| ISBLIND | vak: térkép nem frissül | bájital | időzítő, gyógyítás |
| ISLEVIT | lebeg: csapdák hatástalanok, nem ébreszt, nem lépcsőzhet, nem vehet fel | bájital | időzítő |
| ISHALU | hallucinál: szörnyek/tárgyak véletlen jellel, minden körben újrarajzolva | bájital | időzítő, extra gyógyítás |
| SEEMONST | szörnyek látszanak falon át (inverz), ISINVIS nem | bájital (20 kör) | időzítő |
| ISHASTE | 2 akció / kör | bájital | időzítő |
| ISHELD | nem léphet el a mezőről (harc, tárgyhasználat mehet) | Venus flytrap | flytrap halála, teleport, egyes pálcák flytrapra |

### 1.2.2 Szörny
| flag | jelentés |
|---|---|
| ISMEAN | belépéskor felébredhet (lásd [6.4](06_szornyek.md#64-ébresztés)) |
| ISFLY | ha lépés után dist2(játékos) ≥ 3, még egyszer lép |
| ISREGEN | regenerálódó szörny (griff, troll, vámpír). Fenntartott flag: játékmechanikai hatása nincs |
| ISGREED | előbb a szobabeli aranyra megy és ráül |
| ISINVIS | csak CANSEE mellett látszik |
| ISSLOW | csak minden második körben cselekszik (`turn` váltogat) |
| ISCANC | speciális képesség kikapcsolva |
| ISFOUND | speciális képességet már elsütötte |
| ISTARGET | `f` parancs célpontja |
| ISRUN | üldöz (nélküle a szörny egyáltalán nem mozog) |
| ISHUH | 80% véletlen mozgás; minden zavart lépésnél 1/20 eséllyel feloldódik |
| ISHASTE | 2 akció / kör |
| ISHELD | egyáltalán nem mozog (Hold monster tekercs) |

### 1.2.3 Tárgy
| flag | jelentés |
|---|---|
| ISCURSED | használatban lévő tárgy nem vehető le / nem dobható |
| ISKNOW | azonosítva: bónuszok, töltésszám látszik |
| ISMISL | lövedék (nyíl íjjal, magic missile) |
| ISMANY | kötegelt: nyíl, dobótőr, shuriken, arany |
| ISFOUND | a tárgy már volt a csomagban (minden felvételkor beáll). A scare monster tekercs ebből tudja, hogy a következő felvételkor porrá kell válnia ([8.3](08_targyak.md#83-tekercsek-r)) |
| ISPROT | páncél véd a rozsdától |

### 1.2.4 Mező
`F_REAL` (nem rejtett), `F_PASS` (folyosó vagy labirintus mező), `F_SEEN` (látott), `F_PNUM` (folyosórendszer sorszáma), `F_TMASK` (csapdatípus). Rejtett mező megjelenése: titkos ajtó → az adott fal karaktere (`-` vagy `|`), titkos folyosó → SOLID_ROCK, rejtett csapda → FLOOR.

---

## 1.3 Térkép és képernyő

### 1.3.1 Képernyő
| név | érték |
|---|---|
| NUMCOLS × NUMLINES | 80 × 24 karakter |
| Üzenetsor | 0. sor ([3.4](03_latoter_es_megjelenites.md#34-üzenetsor)) |
| Térkép | 1..22. sor (a 0. és a 23. sorra semmi nem léphet) |
| Státuszsor | 23. sor (`STATLINE = NUMLINES − 1`, [3.3](03_latoter_es_megjelenites.md#33-státuszsor)) |

A térkép két rétegű: a **MAP** a valódi szint (`places[]`: karakter + flagek + szörny-mutató mezőnként), a **SCREEN** az, amit a játékos lát és amire emlékszik (a curses képernyő). Megjelenítéskor a MAP-ből másolunk a SCREEN-re.

### 1.3.2 Karakterek
| konstans | jel | | konstans | jel |
|---|---|---|---|---|
| PASSAGE | `#` | | FOOD | `:` |
| DOOR | `+` | | WEAPON | `)` |
| FLOOR | `.` | | ARMOR | `]` |
| PLAYER | `@` | | AMULET | `,` |
| TRAP | `^` | | RING | `=` |
| STAIRS | `%` | | STICK | `/` |
| GOLD | `*` | | MAGIC | `$` (csak mágia-észlelés) |
| POTION | `!` | | SOLID_ROCK | ` ` |
| SCROLL | `?` | | MONSTER | `A`..`Z` |
| WALL_H | `-` | | WALL_V | `\|` |

Nincs külön sarok-karakter: a felső és alsó fal a sarkokkal együtt `-`, az oldalfalak `|`.
