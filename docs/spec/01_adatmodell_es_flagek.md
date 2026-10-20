# 1. Adatmodell és flagek

## 1.1 Rekordok

### 1.1.1 Room (9 db, szektoronként egy)
```
Room {
  pos: (x,y)          // bal felső sarok (fallal együtt); fantom szobánál maga a folyosó-pont
  size: (w,h)         // méret falakkal együtt
  flags:
    PHANTOM           // "gone" szoba (ISGONE): csak egy folyosó-csomópont
    DARK              // a szoba sötét (ISDARK)
    MAZE              // labirintus (ISMAZE)
  exits[]             // ajtók/kijáratok koordinátái (szörny útkereséshez)
  gold_pos, gold_val  // arany a szobában (ha van)
}
// összekötéshez (csak generálás közben): conn[9][9] szomszédsági tábla, isconn[9][9], ingraph[9]
// folyosórendszerek: passages[] – minden összefüggő folyosóhálózat külön "szobaként" szerepel
//   (r_flags = ISGONE|ISDARK), ez kell a drain life-hoz és a szörnyek útkereséséhez
```

### 1.1.2 Creature (játékos és szörny közös rekord)
```
Creature {
  str       // erő (3..31); szörnynél mindig 10 (6.1)
  exp       // játékos: összegyűjtött tapasztalat; szörny: ennyi XP-t ad, ha megölik (6.2)
  lvl       // szint; játékos: a tapasztalatból (5.2), szörny: típus + mélységi módosító (6.2)
  arm       // páncélosztály: minél kisebb, annál nehezebb eltalálni (játékos alapból 10)
  dmg       // sebzés-string ("NxM/NxM", 0.2); játékosnál fegyver nélküli ütés ("1x4")
  hpt       // aktuális életerő
  maxhp     // maximális életerő
  pos       // pozíció a térképen
  room      // a szoba vagy folyosórendszer, amelyben áll (1.1.1)
  flags     // állapotflagek (1.2.1 játékos, 1.2.2 szörny)
  pack      // list<Item>: játékosnál a csomag, szörnynél a nála lévő tárgyak
  oldch     // a mező képernyőjele a lény alatt; ezt kell visszarajzolni, ha elmegy vagy nem látszik
  // csak szörny:
  type      // 'A'..'Z': a típus betűje, index a típustáblába (6.1)
  name      // a típus neve (6.1), üzenetekhez
  carry     // %: ekkora eséllyel kap tárgyat létrehozáskor (6.3); > 0 esetén tárgyra is vadászhat (6.5)
  dest      // hová tart: pozíció vagy a játékos (6.5)
  disguise  // amilyen jellel kirajzolódik: a saját betűje, a Xeroc álcázva tárgyjel (6.2, 6.7)
  turn      // bool: ISSLOW szörnynél váltogat, csak true-nál lép (6.2)
}
// a Venus flytrap számlálója (vf_hit) nem a lényhez tartozik: globális, a GameState-ben van
```
Játékos-specifikus állapot a `GameState`-ben (lásd lent).

### 1.1.3 Item
```
Item {
  type     // kategória: POTION, SCROLL, FOOD, WEAPON, ARMOR, STICK, RING, GOLD, AMULET
  which    // altípus index a kategórián belül (a 8.2–8.7 táblák # oszlopa); ételnél 0 = ration, 1 = gyümölcs
  pos      // pozíció, ha a padlón van
  packch   // csomagbetű ('a'..'z'), ha a csomagban van (8.9)
  text     // azonosítatlan név: bájitalszín, tekercsfelirat, gyűrűkő, pálcaanyag (8.8)
  launch   // a kilövő fegyver típusa, amellyel lőni kell (nyílnál: short bow); nincs: üres (7.4)
  damage   // sebzés-string közelharcban, kézben tartva ("NxM")
  hurldmg  // sebzés-string dobva vagy kilőve (7.4)
  count    // darabszám kötegelt tárgynál (nyíl, dart, shuriken, dagger, bájital, tekercs, étel)
  hplus    // találati bónusz (fegyvernél; átkozott fegyvernél negatív) (7.1, 8.4)
  dplus    // sebzésbónusz (fegyvernél) (7.1, 8.4)
  arm      // jelentése típusonként: páncélnál védelem (kisebb = jobb); pálcánál töltés; aranynál az összeg; gyűrűnél a mod (pl. protection +1)
  flags    // ISCURSED, ISKNOW, ISMISL, ISMANY, ISFOUND, ISPROT (1.2.3)
  group    // kötegazonosító: azonos which + group → egy csomaghelyen összevonódik (8.4, 8.9)
  label    // a játékos által adott egyedi név ('c'), fegyvernél, páncélnál, amulettnél (4.2.6)
}
```

### 1.1.4 GameState
```
GameState {
  level                  // a jelenlegi dungeon-szint (mélység), 1-től
  max_level              // a legmélyebb eddig elért szint; tárgy/arany csak ott generálódik, ha nálad az amulett (2.7, 2.8)
  gold                   // a játékos aranya; ebből lesz a pontszám (4.3)
  turn                   // eltelt körök száma (nincs használva semmire)
  player                 // Creature: a játékos (1.1.2)
  cur_weapon             // a kézben tartott fegyver (vagy üres)
  cur_armor              // a viselt páncél (vagy üres)
  cur_ring[LEFT]         // gyűrű a bal kézen (vagy üres) (8.6)
  cur_ring[RIGHT]        // gyűrű a jobb kézen (vagy üres)
  food_left              // élelemkészlet: körönként fogy, evéskor nő; 300 alatt Hungry, 150 alatt Weak, 0 alatt ájulás (5.4)
  hungry_state           // 0 = rendben, 1 = Hungry, 2 = Weak, 3 = Faint (5.4)
  no_food                // hány szint óta nem generálódott étel; > 3 esetén a következő tárgy étel (8.1)
  regen_counter          // HP-regeneráció számlálója (5.3)
  vf_hit                 // Venus flytrap fogásszámláló: ennyit sebez a fogva tartás alatt (6.7)
  has_amulet             // a játékosnál van-e az amulett (8.10)
  str_max                // a játékos legnagyobb elért ereje gyűrűk nélkül; a státuszsoron "Str: cur(max)" (5.6)
  timers: {              // hátralévő körök az egyes hatásokból (4.1.1)
    confused             // zavarodottság (ISHUH)
    hallucinating        // hallucináció (ISHALU)
    see_invis            // láthatatlan-látás (CANSEE)
    blind                // vakság (ISBLIND)
    levitating           // lebegés (ISLEVIT)
    hasted               // gyorsítás (ISHASTE)
    no_command           // alvás/ájulás/fagyás: ennyi körig nem cselekedhet a játékos
    bear_trap            // ennyi körig nem léphet el a medvecsapdából (9.1)
    monster_detect       // szörnyérzékelés (SEEMONST)
  }
  wander_timer           // időzítő a vándorló szörny démon indulásáig (6.2)
  wander_check_tick      // a vándorló démon körszámlálója (between, 6.2)
  know[type][which]      // a tárgytípus azonosítva van-e (8.2, 8.3)
  guess[type][which]     // a játékos által a típusnak adott név ('c'), ha még nincs azonosítva
  names[type][which]     // azonosítatlan nevek ebben a játékban: színek, feliratok, kövek, anyagok (8.8)
  stick_is_staff[which]  // az adott pálcatípus ebben a játékban bot (staff) vagy pálca (wand) (8.8)
  rooms[9]               // a 9 szoba (1.1.1)
  monsters               // a szinten lévő szörnyek listája
  floor_items            // a padlón lévő tárgyak listája
  traps                  // a szint csapdái, legfeljebb MAX_TRAPS (2.9); a típus a mező flagjében van (1.2.4) (nincs használva semmire)
  last_command           // az előző parancs, az 'a' ismétléshez (4.2.5)
  last_message           // az utolsó üzenet, a CTRL-P-hez (3.4)
  count_prefix           // a parancs előtti számprefix (pl. "5s") (4.2.5)
  can_huh                // CANHUH: a következő sikeres ütés összezavarja a szörnyet (8.3)
  max_hit_this_fight     // az aktuális harc legnagyobb egyszeri sebzése, az 'f' leállásához (7.2)
}
```

---

## 1.2 Flagek

### 1.2.1 Játékos
| flag | jelentés | be | ki |
|---|---|---|---|
| ISRUN | „ébren van” – egyetlen hatása: ha hiányzik, a szörnyek +4-gyel találják el (a cselekvőképességet a `no_command` számláló kezeli) | alap; ájulás/alvás/fagyás lejártakor | alvás (tekercs, gáz), fagyás, gyorsítás-ájulás, éhség-állapotváltás |
| ISHUH | zavarodott: mozgásnál 80% véletlen irány (`rndmove`, [6.6](06_szornyek.md#66-mozgás): előbb `dy`, aztán `dx = rnd(3)-1`; (0,0) nem visz időt), dobásnál/`^`-nál 20% véletlen irány | bájital, Medusa | időzítő; időtartamok összeadódnak |
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
| flag | jelentés |
|---|---|
| F_PASS | folyosó vagy labirintus mező |
| F_SEEN | a játékos már látta a mezőt |
| F_REAL | nem rejtett („amit látsz, az van ott”); ha hiányzik, a mező rejtett (lásd lent) |
| F_PNUM | a folyosórendszer sorszáma (alsó 4 bit), folyosó-mezőn |
| F_TMASK | a csapda típusa (alsó 3 bit), csapda-mezőn ([9.1](09_csapdak.md#91-rálépés)) |
| F_DROPPED | lerakáskor beáll a mezőn, de a játék semmire nem használja |

*Megjegyzés:* az F_PNUM és az F_TMASK ugyanazokat a biteket használja, ezért egy mező nem lehet egyszerre folyosó és csapda. Emiatt nem kerül csapda labirintusba ([2.9](02_szintgeneralas.md#29-csapdák)).

**Rejtett mező** (F_REAL nélkül) megjelenése, amíg fel nem fedezik:

| mi van ott valójában | minek látszik |
|---|---|
| titkos ajtó | az adott fal karaktere (`-` vagy `\|`) |
| titkos folyosó | SOLID_ROCK (üres) |
| rejtett csapda | FLOOR |

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
