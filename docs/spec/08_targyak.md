# 8. Tárgyak

## 8.1 Generálás
```
new_item():
  if no_food > 3: type = FOOD
  else: r = rnd(100):
     0..25 POTION, 26..61 SCROLL, 62..77 FOOD, 78..84 WEAPON, 85..91 ARMOR, 92..95 RING, 96..99 STICK
  if type == FOOD:
     no_food = 0
     which = (rnd(10) != 0) ? FOOD_RATION : FRUIT          // 90% / 10%
  else:
     which = pick_by_prob(TABLE[type])
  init per type (8.2..8.7)
```
A `no_food` minden új szintnél nő (2), és csak étel *generálásakor* nullázódik; a kényszerített étel a `no_food > 3` utáni első tárgygenerálás (kincses szobában is).

## 8.2 Bájitalok (`q`)
| # | név | esély | érték | hatás |
| --- | --- | --- | --- | --- |
| 0 | confusion | 7 | 5 | ISHUH, +spread(20) kör (ha nem hallucinál, üzenet) |
| 1 | hallucination | 8 | 5 | ISHALU, spread(850) |
| 2 | poison | 8 | 5 | ha nincs sustain strength: chg_str(-(rnd(3)+1)), „you feel very sick now”, és a hallucináció is megszűnik (`come_down`); sustain strength-tel: „you feel momentarily sick” |
| 3 | gain strength | 13 | 150 | chg_str(+1) |
| 4 | see invisible | 3 | 100 | CANSEE, spread(850) ([4.1.1](04_jatekciklus_es_parancsok.md#411-időzített-hatások-fuse-ok)); a vakság is megszűnik; „this potion tastes like <fruit> juice” |
| 5 | healing | 13 | 130 | hpt += ROLL(lvl,4); ha > maxhp: maxhp++, hpt = maxhp; vakság le |
| 6 | monster detection | 6 | 130 | SEEMONST 20 kör |
| 7 | magic detection | 6 | 105 | mágia-képernyő ([3.2](03_latoter_es_megjelenites.md#32-állapotok-hatása-a-kijelzésre)) |
| 8 | raise level | 2 | 250 | exp = xp_next[lvl-1] + 1; szintlépés |
| 9 | extra healing | 5 | 200 | hpt += ROLL(lvl,8); ha > maxhp: (hpt > maxhp+lvl+1 ? maxhp+=2 : maxhp+=1); hpt = maxhp; vakság, hallucináció le |
| 10 | haste self | 5 | 190 | ha már ISHASTE: hatás le, ájulás rnd(8) kör; különben ISHASTE rnd(4)+4. Az ivás nem visz el időt |
| 11 | restore strength | 13 | 130 | str = str_max |
| 12 | blindness | 5 | 5 | ISBLIND spread(850) |
| 13 | levitation | 6 | 75 | ISLEVIT spread(30) |

**Automatikus azonosítás iváskor:** azonosítódik: poison, gain strength, healing, extra healing, raise level, haste self, hallucination, blindness, levitation; confusion csak ha a játékos nem hallucinál; magic detection csak ha mutatott valamit. **Nem** azonosítódik: see invisible, monster detection, restore strength. Ha a típus ivás után sem ismert és nincs még elnevezve, a játék megkérdezi: „what do you want to call it?” (`call_it`). Ha viszont ismert, a korábbi `c`-név törlődik (azonosító tekercsnél is). Ugyanez a tekercseknél.

Nem megfelelő tárgyra a `q`, `r` és `e` is időbe kerül („yuk! Why would you want to drink that?”, „there is nothing on it to read”, „ugh, you would get ill if you ate that”).

## 8.3 Tekercsek (`r`)
| # | név | esély | érték | hatás |
| --- | --- | --- | --- | --- |
| 0 | monster confusion | 7 | 140 | CANHUH; „your hands begin to glow red” |
| 1 | magic mapping | 4 | 150 | a térkép (padló, tárgyak, szörnyek nélkül) a SCREEN-re: falak, ajtók, folyosók, lépcső; a titkos ajtók ajtóvá, a titkos folyosók folyosóvá, a rejtett csapdák látható csapdává válnak |
| 2 | hold monster | 2 | 180 | a játékos körüli 5×5-ben az ISRUN szörnyek: ISRUN le, ISHELD be; ha volt ilyen: „the monster(s) freeze(s)” és azonosítódik, különben „you feel a strange sense of loss” |
| 3 | sleep | 3 | 5 | játékos alszik rnd(5)+4 kör (ISRUN le) |
| 4 | enchant armor | 7 | 160 | viselt páncél: arm--, ISCURSED le; páncél nélkül nincs hatás és üzenet sem |
| 5 | identify potion | 10 | 80 | lásd lent |
| 6 | identify scroll | 10 | 80 | lásd lent |
| 7 | identify weapon | 6 | 80 | lásd lent |
| 8 | identify armor | 7 | 100 | lásd lent |
| 9 | identify ring/stick | 10 | 115 | lásd lent |
| 10 | scare monster | 3 | 200 | olvasva: „you hear maniacal laughter in the distance”. Padlón: szörny nem lép rá, nem születik rá. Az **első** felvételkor ISFOUND jelet kap; ha valaha letették, a következő felvételkor „the scroll turns to dust as you pick it up” |
| 11 | food detection | 2 | 60 | ételek helye külön képernyőn („Your nose tingles and you smell food.--More--”); ha nincs: „your nose tingles” |
| 12 | teleportation | 5 | 165 | teleport(); csak akkor azonosítódik, ha más szobába került |
| 13 | enchant weapon | 8 | 150 | kézben lévő fegyver: ISCURSED le; 50% hplus++, 50% dplus++; fegyver nélkül „you feel a strange sense of loss” |
| 14 | create monster | 4 | 75 | new_monster(level, INITIAL) egy véletlen szabad szomszéd mezőre (scare monster tekercsre nem); a szörny alszik és nem kap tárgyat; nincs hely: „you hear a faint cry of anguish in the distance” |
| 15 | remove curse | 7 | 105 | páncél, fegyver, két gyűrű ISCURSED le |
| 16 | aggravate monsters | 3 | 20 | minden szörny a szinten: runto (ISRUN, ISHELD le); „you hear a high pitched humming noise” |
| 17 | protect armor | 2 | 250 | viselt páncél ISPROT; páncél nélkül „you feel a strange sense of loss” |

**Automatikus azonosítás olvasáskor:** mindig: magic mapping, sleep és az öt identify tekercs. Feltétellel: hold monster, ha fagyasztott; food detection, ha volt étel; teleportation, ha más szobába került. **Soha:** monster confusion, enchant armor, scare monster, enchant weapon, create monster, remove curse, aggravate monsters, protect armor. Olvasás után `look(TRUE)` fut, így a szomszédos szörnyek felébredhetnek.

**Azonosító tekercsek:** maga a tekercs azonosítódik („this scroll is an identify … scroll”). Üres csomagnál: „you don't have anything in your pack to identify”. Ezután addig kérdez, amíg a típusnak megfelelő tárgyat nem választ: rossz típusra „you must identify a <potion | scroll | weapon | suit of armor | ring, wand or staff>”, ESC-re „you must identify something”. Csak akkor lép ki eredmény nélkül, ha a játékos `*`-ot nyom és a szűrt lista üres. A kiválasztott tárgy ISKNOW-t kap, bájital/tekercs/gyűrű/pálca esetén a típusa is ismertté válik (az esetleges `c`-név törlődik), majd megjelenik a tárgy neve.

```
teleport():
  P = find_floor(null, monster)   // bármely nem fantom szoba; lehet ugyanaz a szoba, akár ugyanaz a mező is
  ha más szobába került: leave_room + enter_room, különben look()
  player.pos = P
  if ISHELD: ISHELD le; vf_hit = 0
  bear_trap (no_move) = 0; futás/ismétlés megszakad; billentyűpuffer ürítése
```

## 8.4 Fegyverek
A fegyvertípusok:

| # | név | esély | érték | kézben | dobva | megjegyzés |
|---|---|---|---|---|---|---|
| 0 | mace | 11 | 8 | 2x4 | 1x3 | |
| 1 | long sword | 11 | 15 | 3x4 | 1x2 | |
| 2 | short bow | 12 | 15 | 1x1 | 1x1 | kilövő |
| 3 | arrow | 12 | 1 | 1x1 | 2x3 | count rnd(8)+8, ISMANY, ISMISL, launch = bow |
| 4 | dagger | 8 | 3 | 1x6 | 1x4 | count rnd(4)+2 |
| 5 | two handed sword | 10 | 75 | 4x4 | 1x2 | |
| 6 | dart | 12 | 2 | 1x1 | 1x3 | count rnd(8)+8, ISMANY |
| 7 | shuriken | 12 | 5 | 1x2 | 2x4 | count rnd(8)+8, ISMANY |
| 8 | spear | 12 | 5 | 2x3 | 1x6 | |

```
init_weapon(w):
  r = rnd(100)
  if r < 10:  w.ISCURSED; w.hplus -= rnd(3)+1
  else if r < 15: w.hplus += rnd(3)+1
  // dplus = 0
  ISMANY tárgyaknál és a daggernél group = új egyedi azonosító (a dagger nem ISMANY, mégis kötegel)
```

## 8.5 Páncélok
A páncéltípusok:

| # | név | esély | érték | védelem |
|---|---|---|---|---|
| 0 | leather armor | 20 | 20 | 8 |
| 1 | ring mail | 15 | 25 | 7 |
| 2 | studded leather | 15 | 20 | 7 |
| 3 | scale mail | 13 | 30 | 6 |
| 4 | chain mail | 12 | 75 | 5 |
| 5 | splint mail | 10 | 80 | 4 |
| 6 | banded mail | 10 | 90 | 4 |
| 7 | plate mail | 5 | 150 | 3 |

```
init_armor(a):
  a.arm = base
  r = rnd(100)
  if r < 20: a.ISCURSED; a.arm += rnd(3)+1
  else if r < 28: a.arm -= rnd(3)+1
```
Átkozott páncél nem vehető le; remove curse után levehető, a rossz érték marad.

## 8.6 Gyűrűk (`P` fel, `R` le)
| # | név | esély | étel/kör | érték | hatás |
| --- | --- | --- | --- | --- | --- |
| 0 | protection | 9 | +1 | 400 | páncél −arm, mentődobás + |
| 1 | add strength | 9 | +1 | 400 | str += mod felhúzáskor, −mod levételkor ([5.6](05_jatekos.md#56-erő)) |
| 2 | sustain strength | 5 | +1 | 280 | véd méreg/kígyó erővesztés ellen |
| 3 | searching | 10 | −3 | 420 | minden körben search() |
| 4 | see invisible | 10 | −5 | 310 | CANSEE |
| 5 | adornment | 1 | 0 | 10 | – |
| 6 | aggravate monster | 10 | 0 | 10 | mindig átkozott; felhúzáskor aggravate; új szörnyek ISRUN |
| 7 | dexterity | 8 | −3 | 440 | találat + arm (csak kézben tartott fegyverrel) |
| 8 | increase damage | 8 | −3 | 400 | sebzés + arm (csak kézben tartott fegyverrel) |
| 9 | regeneration | 4 | +2 | 460 | +1 HP/kör |
| 10 | slow digestion | 9 | −2 | 240 | 50%: −1 fogyás |
| 11 | teleportation | 5 | 0 | 30 | mindig átkozott; körönként rnd(50)==0 → teleport() |
| 12 | stealth | 7 | +1 | 470 | nem ébreszt |
| 13 | maintain armor | 5 | +1 | 380 | véd Aquator ellen |

```
init_ring(r):
  if which in {protection, add strength, dexterity, increase damage}:
     r.arm = rnd(3); if r.arm == 0: r.arm = -1; r.ISCURSED
  if which in {aggravate, teleportation}: r.ISCURSED
```
Két kéz (bal/jobb). Átkozott nem vehető le.

## 8.7 Pálcák/botok (`z`)
| # | név | esély | érték | hatás |
| --- | --- | --- | --- | --- |
| 0 | light | 12 | 250 | sötét szobában DARK le, szoba kirajzolva; folyosón „the corridor glows and then fades” |
| 1 | invisibility | 6 | 5 | első szörny a sugár útjában: ISINVIS |
| 2 | lightning | 3 | 330 | fire_bolt "bolt" |
| 3 | fire | 3 | 330 | fire_bolt "flame" |
| 4 | cold | 3 | 330 | fire_bolt "ice" |
| 5 | polymorph | 15 | 310 | első szörny: `new_monster` a 26 típus közül egyenletesen (a régi típus is lehet), teljesen új statokkal és flagekkel – ezért **elalszik** (nincs ISRUN); tárgyait megtartja; csak akkor azonosítódik, ha az új szörny látható |
| 6 | magic missile | 10 | 170 | lövedék, a dobott tárgy mozgásával (falig/szörnyig); ha szörnyet ér és az !save_throw_magic: dobott támadás +100 találattal (gyakorlatilag mindig talál), sebzés 1d4 + 1 + add_dam[str] + kézben lévő fegyver dplus (a fegyver hplus-a is hozzáadódik). Egyébként „the missle vanishes with a puff of smoke”. **Kézben tartott fegyver nélkül a lövedék nem sebez** (a lövedéknek csak dobott sebzése van, az pedig csak kilövő fegyverrel érvényes; kézi sebzése üres), de a szörny felébred |
| 7 | haste monster | 10 | 5 | ISRUN; ha ISSLOW: le, különben ISHASTE |
| 8 | slow monster | 11 | 350 | ISRUN; ha ISHASTE: le, különben ISSLOW |
| 9 | drain life | 9 | 300 | ha hpt < 2: „you are too weak to use it”, nincs töltésfogyás. Célpontok: szobában a szoba összes szörnye; folyosón az **egész összefüggő folyosórendszer** szörnyei és az ahhoz tartozó ajtókon állók; ajtón állva a szoba és a folyosó is. Ha nincs célpont: „you have a tingling feeling” (a töltés fogy). Ha van: `hpt /= 2`; minden célpont `hpt / n`-t veszít (a *maradék* hp-ból számolva), aki túléli, runto |
| 10 | nothing | 1 | 5 | – |
| 11 | teleport away | 6 | 340 | első szörny: find_floor(null, monster), de nem a játékos mezője; ISRUN, dest = játékos |
| 12 | teleport to | 6 | 50 | első szörny: a játékos melletti mezőre a lövés irányában (ez mindig szabad, mert a sugár azon át haladt, vagy maga a szörny áll ott); ISRUN, dest = játékos |
| 13 | cancellation | 5 | 280 | első szörny: ISCANC, ISINVIS le, xeroc álca le |

```
init_stick(s):
  s.arm (charges) = (which == light) ? rnd(10)+10 : rnd(5)+3
  if stick_is_staff[which]: s.damage = "2x3" else s.damage = "1x1"
  s.hurldmg = "1x1"

zap():
  s = ask_item(STICK); dir = ask_direction (ha kell)
  if s.arm == 0: "nothing happens"; return TIME      // a pálca nem tűnik el
  hatás; s.arm--
  "első szörny az útban": a sugár a játékostól indul dir irányba, falig/ajtóig; az első szörnyön megáll
  ha a célpont Venus flytrap (invisibility/polymorph/teleport away/to/cancellation): player.ISHELD le
```
- Nem pálcára: „you can't zap with that!” (nem visz időt).
- Azonosítódik: light (folyosón is), magic missile, lightning/fire/cold mindig; polymorph, ha az új szörny látható. Nem azonosítódik: invisibility, haste/slow monster, drain life, nothing, teleport away/to, cancellation.
- Az invisibility, polymorph és cancellation nem ébreszti fel a szörnyet. A teleport away/to ISRUN-t ad, de a Hold monster okozta ISHELD-et nem törli.

## 8.8 Azonosítás és nevek
Játék indulásakor minden típus kap egy véletlen, a játék végéig rögzített „álnevet”, ismétlés nélkül. Az álnevek listái:

| | lista |
|---|---|
| **Bájital színek** (27, ebből 14 kell) | amber, aquamarine, black, blue, brown, clear, crimson, cyan, ecru, gold, green, grey, magenta, orange, pink, plaid, purple, red, silver, tan, tangerine, topaz, turquoise, vermilion, violet, white, yellow |
| **Gyűrű kövek** (26, ebből 14 kell) – a kő értéke hozzáadódik a gyűrű értékéhez | agate 25, alexandrite 40, amethyst 50, carnelian 40, diamond 300, emerald 300, germanium 225, granite 5, garnet 50, jade 150, kryptonite 300, lapis lazuli 50, moonstone 50, obsidian 15, onyx 60, opal 200, pearl 220, peridot 63, ruby 350, sapphire 285, stibotantalite 200, tiger eye 50, topaz 60, turquoise 70, taaffeite 300, zircon 80 |
| **Bot – fa** (33) | avocado wood, balsa, bamboo, banyan, birch, cedar, cherry, cinnibar, cypress, dogwood, driftwood, ebony, elm, eucalyptus, fall, hemlock, holly, ironwood, kukui wood, mahogany, manzanita, maple, oaken, persimmon wood, pecan, pine, poplar, redwood, rosewood, spruce, teak, walnut, zebrawood |
| **Pálca – fém** (22) | aluminum, beryllium, bone, brass, bronze, copper, electrum, gold, iron, lead, magnesium, mercury, nickel, pewter, platinum, steel, silver, silicon, tin, titanium, tungsten, zinc |

**Tekercsfelirat-szótagok:**
```
a ab ag aks ala an app arg arze ash bek bie bit bjor blu bot bu byt comp con cos cre dalf dan den do e eep el eng er ere erk esh evs fa fid fri fu gan gar glen gop gre ha hyd i ing ip ish it ite iv jo kho kli klis la lech mar me mi mic mik mon mung mur nej nelg nep ner nes nes nih nin o od ood org orn ox oxy pay ple plu po pot prok re rea rhov ri ro rog rok rol sa san sat sef seh shu ski sna sne snik sno so sol sri sta sun ta tab tem ther ti tox trol tue turs u ulk um un uni ur val viv vly vom wah wed werg wex whon wun xo y yot yu zant zeb zim zok zon zum
```

```
init_materials():                 // bot vagy pálca típusonként
  for each stick type:
    loop:
      if rnd(2) == 0: j = rnd(len(metal)); if unused: type = "wand", anyag = metal[j]; break
      else:           j = rnd(len(wood));  if unused: type = "staff", anyag = wood[j]; break

init_names():                     // tekercs feliratok
  for each scroll type:
    rnd(3)+2 szó, mindegyik rnd(3)+1 szótagból a szótag-listából, max. 40 karakter
    szótagok: a fenti tekercsfelirat-szótagok
```

**Tárgynevek:**
| állapot | bájital / gyűrű / pálca | tekercs |
|---|---|---|
| ismert | `A potion of healing(blue)`, `2 potions of …`; gyűrűnél `A ring of protection [+1](ruby)`; pálcánál `A wand of light [5 charges](iron)` | `A scroll of identify potion` |
| elnevezett (`c`) | `A potion called <név>(blue)` | `A scroll called <név>` |
| ismeretlen | `A blue potion`, `2 blue potions` | `A scroll titled 'zok bie'` |

Fegyver: `A mace`, azonosítva `A +1,+1 mace`, köteg: `25 +0,+0 arrows`. Páncél: `ring mail`, azonosítva `+1 ring mail [protection 4]` (terse módban `[4]`; a szám `10 − arm`, az előjeles szám `alap − arm`). Étel: `Some food` / `3 rations of food`, gyümölcs `A slime-mold` / `2 slime-molds`. Arany: `25 Gold pieces`. Amulett: `The Amulet of Yendor`. A csomaglistában a használt tárgyak után: `(weapon in hand)`, `(being worn)`, `(on left hand)` / `(on right hand)`.

- Fegyver/páncél bónusza, pálca töltése, gyűrű modja csak ISKNOW esetén látszik.

**Mágikus tárgy** (magic detection, nymph): potion, scroll, stick, ring, amulet mindig; páncél ha ISPROT vagy módosított (`arm != base`); fegyver ha `hplus != 0 or dplus != 0`.

## 8.9 Csomag
```
pick_up(it):
  if GOLD: gold += it.arm; "you found %d gold pieces"; remove; return    // nem kerül a csomagba
  if it is scare monster and it.ISFOUND: "the scroll turns to dust as you pick it up"; remove; return
  // helyszámlálás: inpack = a csomag DARABSZÁMA, kivéve a csoportos (group) fegyvereket
  //   – 3 egyforma bájital 3 helyet foglal; egy 25-ös nyílköteg 1 helyet
  if inpack + 1 > MAX_PACK (23) (köteghez csatolásnál nem számít bele): "there's no room in your pack"; return
  merge:
    POTION/SCROLL/FOOD azonos which → count++
    fegyver azonos which és group → count += it.count
  else: új bejegyzés, it.packch = legkisebb szabad betű 'a'..'z'; a többi betű nem csúszik
        a csomag típus szerint rendezett: az új tárgy az azonos típusúak mellé kerül
  it.ISFOUND = true
  minden szörny, amelyik erre a tárgyra tartott: dest = player
  if AMULET: has_amulet = true
```
Lerakás (`d`): csak ha a mező FLOOR vagy PASSAGE (különben „there is something there already”); az Amulett is lerakható (ekkor `has_amulet = false`).

## 8.10 Arany, Amulett, Étel
- Arany: `arm` = mennyiség; nem foglal csomaghelyet.
- Amulett: `,`; az `AMULET_LEVEL` (26.) szinttől generálódik ([2.8](02_szintgeneralas.md#28-tárgyak-put_things)); megszerzésekor `has_amulet`; felfelé lépcsőzés engedélyezett; éhség alapfogyása 0. Lerakható, eldobható, és a nimfa is ellophatja (mágikus tárgy).
- Étel: két fajta: food ration (90%) és gyümölcs (10%, alapnév „slime-mold”) Mindig 1 darab generálódik.
