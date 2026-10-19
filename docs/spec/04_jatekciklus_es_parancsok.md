# 4. Játékciklus és parancsok

## 4.1 Fő játékciklus

```
main():
  init_identification_tables()     // 8.8
  init_player()                    // 5.1
  level = 1; new_level()
  loop: command()

command():                          // egy kör
  // BEFORE fázis
  wander_daemon()                  // 6.2 (rollwand)
  if ISHALU: hallucination_redraw()
  BEFORE-időzítők ketyegése
  actions = (player ISHASTE) ? 2 : 1
  repeat actions:
    look(wakeup=true)              // látótér frissítés + szomszédok ébresztése (3.1, 6.4)
    update_status(); draw()
    if no_command > 0:             // alszik / ájult / fagyott
       cmd = '.'                   // pihenésnek számít
       if --no_command == 0: player.ISRUN = true; "you can move again"
    else:
       cmd = futás/ismétlés/to_death esetén az előző, különben get_command()
    took_time = execute(cmd)
    if not took_time: actions++    // ingyenes parancs: nem fogy az akció
    ha a lépés tárgyra vitt: pick_up
  // AFTER fázis
  move_monsters()                  // 6.6 (runners)
  player_regen()                   // 5.3 (doctor)
  hunger()                         // 5.4 (stomach)
  AFTER-időzítők ketyegése (4.1.1)
  for hand in LEFT, RIGHT:         // gyűrűk (8.6)
    if searching gyűrű: search()
    else if teleportation gyűrű and rnd(50) == 0: teleport()
```
A `waste_time()` ([4.2.3](#423-tárgyak)) a BEFORE és AFTER fázis démonjait/időzítőit futtatja le még egyszer, játékos-akció nélkül.

### 4.1.1 Időzített hatások (fuse-ok)
| hatás | időtartam | lejáratkor |
|---|---|---|
| confusion | spread(20) (összeadódik) | ISHUH le, „you feel less confused now” |
| hallucination | spread(850) | ISHALU le, képernyő újrarajzolás |
| see invisible (bájital) | spread(850) | CANSEE le – feltétel nélkül: ha a bájital hatása alatt see invisible gyűrűt húzott fel, a gyűrű ellenére is. Ha iváskor már volt CANSEE (pl. gyűrűtől), nem indul időzítő |
| blindness | spread(850) | ISBLIND le, képernyő frissítés |
| levitation | spread(30) | ISLEVIT le |
| haste self | rnd(4)+4 | ISHASTE le |
| monster detection | 20 | SEEMONST le, inverz szörnyek eltűnnek |
| sleep (tekercs) | rnd(5)+4 | ISRUN vissza |
| sleeping gas | 5 | ISRUN vissza |
| bear trap | 3 | – |
| ájulás (haste ütközés) | rnd(8) | ISRUN vissza |
| ájulás (éhség) | rnd(8)+4 | ISRUN vissza |
| fagyás (Ice monster) | rnd(2)+2 összeadódik | ISRUN vissza |

---

## 4.2 Parancsok

`*` = nem visz el időt. Minden más 1 kör (ha sikeres). Ismeretlen billentyű: „illegal command”.

### 4.2.1 Mozgás
| parancs | leírás |
|---|---|
| h j k l y u b n | lépés; szörnyre lépés = támadás; tárgyra lépve automatikus felvétel |
| m + irány | lépés felvétel nélkül („moved onto …”) |
| SHIFT + irány | futás (lásd lent) |
| CTRL + irány | óvatos futás: ezen felül ajtó mellett, elágazásnál és bármi érdekes mellett is megáll (lásd lent); vakon ugyanaz, mint a SHIFT |
| `>` * | lefelé lépcsőn (ha a lépcsőn áll és nem lebeg) |
| `<` * | felfelé lépcsőn (ha a lépcsőn áll, nem lebeg, és `has_amulet`) |

**Mozgás szabályai:**
```
move(dir):
  if bear_trap > 0: bear_trap--; "you are still stuck in the bear trap"; uses turn
  if ISHUH and rnd(5) != 0:
     T = rndmove(player)          // 6.6: ha a véletlen cél nem járható (fal, kő, szabálytalan átló,
     if T == pos: no time; futás és 'f'-harc vége; return   //  szörny, scare monster tekercs), helyben marad
  else: T = pos + dir
  if T a pályán kívül (0. sor, 23. sor, oszlop < 0 vagy ≥ 80): mint falnál (lent)
  átlós lépés tilos, ha bármelyik ortogonális szomszéd fal/kő (sarokvágás tilalom) – ajtón átlósan sem lehet ki/be lépni; no time
  if T rejtett csapda és !ISLEVIT: felfedődik (TRAP, F_REAL be)
  else if ISHELD and T-n nem a flytrap jele áll: "you are being held"; uses turn
  if monster at T: attack(T)
  else if T walkable (FLOOR, DOOR, PASSAGE, STAIRS, TRAP, tárgy):
     pos = T; check room enter/leave (3.1, 6.4)
     if TRAP: trigger (9)
     if item and not 'm' and not ISLEVIT: pick_up
  else: no time
```
A zavarodott véletlen lépés soha nem támad szörnyre (a szörny jele nem járható), és Scare monster tekercsre sem lép.

**Futás:** a futás ugyanazt a lépést ismétli minden akcióban, amíg valami le nem állítja.

*Mindkét futást megállítja:*
- fal, kő vagy a pálya széle (kivéve a kanyarkövetést, lent), szabálytalan átlós lépés;
- ajtóra lépés, lépcsőre, tárgyra vagy szörnyre lépés (szörnynél harc indul);
- csapdára lépés, teleport;
- ha egy szörny megtámadja a játékost, vagy a sárkány tüzet okád;
- éhség-állapotváltás ([5.4](05_jatekos.md#54-éhség));
- ha a keresés (searching gyűrű) talál valamit;
- confusion-nál, ha a véletlen irány helyben hagyná.

*Kanyarkövetés (`passgo` opció, alapból KI):* folyosón, ha a futás falba ütközne és pontosan egy irányba lehet derékszögben továbbmenni, a futás befordul. Csak ortogonális futásnál, vakságban nem.

*CTRL-futás (`door_stop`) – minden lépés után megnézi a 3×3 környezetet, a futás irányával ellentétes oldalt kihagyva:*
- egy ajtó a futás vonalában (ugyanabban a sorban/oszlopban) → megáll;
- több mint egy folyosó-mező oldalirányban (elágazás) → megáll;
- bármi más, ami nem padló, fal vagy üres (tárgy, lépcső, szörny, csapda) → megáll;
- az első lépés után kezd ellenőrizni (az indulóhelyet nem nézi).

*A SHIFT-futást egy szörny megjelenése önmagában nem állítja meg*, csak a támadása.

### 4.2.2 Harc és célzás
| parancs | leírás |
|---|---|
| f + irány | harc a célponttal, amíg: valaki meg nem hal, a célpont elmozdul, más szörny támad, vagy `hpt <= max_hit_this_fight`. Csak látott (vagy SEEMONST-tal érzékelt) szörnyre, különben „I see no monster there” (nincs idő). Szabálytalan átlós célnál semmi nem történik, de a kör elmegy. Nincsenek hit/miss üzenetek. |
| SHIFT-f + irány | ugyanez, HP-ellenőrzés nélkül („kamikaze”) |
| t + irány | hajítás ([7.4](07_harc.md#74-hajítás-t)) |
| z + irány | pálca/bot használata ([8.7](08_targyak.md#87-pálcákbotok-z)) |

### 4.2.3 Tárgyak
| parancs | leírás |
|---|---|
| `,` | felvétel (lebegve nem) |
| d | lerakás – csak üres FLOOR vagy PASSAGE mezőre; használatban lévő átkozottat nem |
| q | bájital ivás |
| r | tekercs olvasás |
| e | evés |
| w | fegyver kézbe (átkozott jelenlegi fegyvert nem lehet cserélni). Bármilyen nem-páncél tárgy kézbe vehető (bájital, étel …), csak a páncélt utasítja el |
| SHIFT-w | páncél felvétele – 2 kör (`waste_time`); a páncél ezzel azonosítódik (ISKNOW). Ha már van rajta páncél: „you are already wearing some. You'll have to take it off first”, nem visz időt. Nem páncélra: „you can't wear that” (időbe kerül) |
| SHIFT-t | páncél levétele – 2 kör (`waste_time`); átkozottat nem: „you can't. It appears to be cursed”. Ha nincs rajta: „you aren't wearing any armor”, nem visz időt |
| SHIFT-p | gyűrű felhúzása (melyik kézre kérdez, ha mindkettő szabad) |
| SHIFT-r | gyűrű levétele (átkozottat nem) |
| c * | tárgytípus elnevezése (bájital, tekercs, gyűrű, pálca) |

**Időveszteség (`waste_time`):** a parancs saját köre előtt még egy teljes „üres” kör lefut, amelyben a játékos nem cselekszik, de minden más igen: a szörnyek lépnek és támadnak, fogy az étel, regenerál a HP, ketyegnek az időzítők (a [4.1](#41-fő-játékciklus) BEFORE és AFTER fázisa). A gyűrűhatások (searching, teleportation) ebben az üres körben **nem** futnak le. A páncél felvétele és levétele így összesen 2 körbe kerül. Ugyanez történik, ha a viselt páncélt más parancs veszi le automatikusan, mert lerakod (`d`) vagy eldobod (`t`).
```
waste_time():
  BEFORE-démonok (vándorló-szörny dobás, hallucináció); BEFORE-időzítők
  AFTER-démonok (szörnymozgás, regeneráció, éhség); AFTER-időzítők
```

### 4.2.4 Információ (mind *)
| parancs | leírás |
|---|---|
| i | teljes csomag listája ([4.2.6](#426-tárgyválasztás-listák-és-tárgyparancsok)) |
| SHIFT-i | egy tárgy betű alapján a státusz/üzenetsorba; üres csomag: „you aren't carrying anything”; 1 tárgy → azonnal; rossz betű: „'x' not in pack” |
| `)` `]` `=` | kézben lévő fegyver / viselt páncél / gyűrűk; „you are wielding nothing” / „wearing nothing” |
| `@` | státuszsor kiírása az üzenetsorba |
| SHIFT-d | felfedezett típusok listája ([4.2.6](#426-tárgyválasztás-listák-és-tárgyparancsok)) |
| `/` | jel magyarázata ([4.2.6](#426-tárgyválasztás-listák-és-tárgyparancsok)) |
| `^` + irány | csapda azonosítása a szomszéd mezőn („You have found <csapda>”), rejtettnél vagy ha nincs: „You have found no trap there” (terse: csak „no trap there”). ISHUH-nál 20% eséllyel más irány (mint minden irányos parancsnál, `get_dir`); ISHALU alatt véletlen csapdanév |
| CTRL-p | utolsó üzenet |
| `?` | súgó ([4.2.6](#426-tárgyválasztás-listák-és-tárgyparancsok)) |
| v | verzió („version 5.4.4. (mctesq was here)”) |

### 4.2.5 Rendszer
| parancs | leírás |
|---|---|
| `.` | pihenés 1 kör |
| s | keresés ([9.2](09_csapdak.md#92-keresés-s-searching-gyűrű)) |
| a | utolsó parancs ismétlése; „you haven't typed a command yet” |
| szám + parancs | ismétlés ≤ 255-ször. Megmarad: `.` `a` `s` `t` `q` `r` `z` `m`, a mozgások (kis- és nagybetű, CTRL), `C`, `I`; minden másnál a szám elvész. Megszakítja (a futással azonos esetek): szörnytámadás, sárkánytűz, éhség-állapotváltás, csapda, teleport, sikeres keresés, illegális parancs |
| Esc * | megszakít |
| o * | beállítások (lásd lent) |
| SHIFT-s * | mentés: „save file (<név>)?” (Y/N), vagy „file name: ” bekérése; mentés után a program kilép. Betöltéskor a mentésfájl törlődik (nem lehet ugyanonnan újrakezdeni) |
| SHIFT-q * | kilépés: „really quit?” – `y`-ra „You quit with <n> gold pieces”, pontszám = arany (levonás nélkül) |
| CTRL-r * | újrarajzolás |
| `!` | alshell indítása; kilépés után a játék folytatódik |
| szóköz * | nem csinál semmit |

**Beállítások (`o`):**
| opció | alapérték | jelentés |
|---|---|---|
| terse | ki | rövidebb üzenetek; a dobott tárgy repülése nem animálódik |
| flush | ki | harcban eldobja az előre leütött billentyűket |
| jump | ki | futás közben nem rajzol minden lépést, csak a végén |
| seefloor | be | sötét szobában a 3×3 környezet padlója látszik ([3.1](03_latoter_es_megjelenites.md#31-felderítés)) |
| passgo | ki | futásnál a folyosó kanyarjait követi ([4.2.1](#421-mozgás)) |
| tombstone | be | halálkor sírkő-rajz ([4.3](#43-szintváltás-és-játék-vége)) |
| inven | clear (ha a terminál tud sorvégig törölni, egyébként overwrite) | csomaglista: overwrite (a térkép fölé), slow (soronként az üzenetsorban), clear (külön képernyő) |
| name | a felhasználó neve | a sírkövön és a ranglistán |
| fruit | slime-mold | a gyümölcs neve |
| file | `<home>/rogue.save` | a mentésfájl neve |

≤ 1200 baudos terminálon a terse és a jump alapból be, a seefloor ki. Az opciók induláskor a `ROGUEOPTS` környezeti változóból is megadhatók (vesszővel elválasztva, pl. `terse,nojump,name=Bob,fruit=apple`; a logikai opciók `no` előtaggal kapcsolhatók ki). Az opciónév rövidíthető (előtag-egyezés); a `~`-mal kezdődő szöveges érték a home könyvtárra bővül; az `inven` értéke a három név (overwrite, slow, clear) előtagja lehet.

**Mentésfájl:**
- Tartalma sorrendben: a verziószöveg, a képernyőméret („<sorok> x <oszlopok>”), majd a teljes játékállapot: minden globális változó és flag, az opciók, a tárgytípusok neve/ismertsége/elnevezése, a szintek számlálói (`level`, `max_level`, `no_food`, `food_left`, `purse`, `vf_hit`, `seed` …), a térkép (`places[]`), a szobák és folyosórendszerek, a szörnyek a csomagjaikkal, a padlón lévő tárgyak, a játékos csomagja, a futó démonok és időzítők.
- Minden bájt egy rögzített kulcsszöveggel és egy ismert állandó szöveggel XOR-olva, futó visszacsatolással kerül a fájlba (egyszerű elrejtés, nem biztonsági titkosítás).
- Mentés után a fájl csak olvasható lesz (0400), a program kilép.
- Betöltéskor elutasítja: más verzió („Sorry, saved game is out of date.”), nagyobb eredeti képernyő, szimbolikus vagy több linkkel rendelkező fájl („Cannot restore from a linked file”), halott játékos („"He's dead, Jim"”). A fájlt a verzió- és képernyőméret-ellenőrzés után, de a link- és a „halott”-ellenőrzés **előtt törli**, így az ezeken elbukó mentés is elvész. Sikeres betöltés után „file name: <fájl>” üzenettel folytatja a játékot.
- A terminál bontásakor (SIGHUP) és programhibát jelző szignáloknál a játék automatikusan ment.

### 4.2.6 Tárgyválasztás, listák és tárgyparancsok

**Tárgy bekérése (`get_item(cél, típus)`):**
```
if pack üres: "you aren't carrying anything"; return null
if 'a' ismétlés: az előző kiválasztott tárgy (ha elfogyott: "you ran out")
loop:
  "which object do you want to <cél>? (* for list): "     (terse: "<cél> what? (* for list): ")
  ESC → "" (törlés), return null, nem visz időt
  '*' → a csomag szűrt listája (inventory, a típusnak megfelelők), utána újra kérdez;
        ha nincs megfelelő: "you don't have anything appropriate", return null (nem visz időt)
  betű → ha nincs ilyen: "'<x>' is not a valid item", újra; különben return a tárgy
```
A típus csak a `*` listát szűri – bármelyik betű elfogadott; a hibás típust maga a parancs utasítja el.

**Csomaglista (`i`, és a `*` lista):** soronként `<betű>) <név>` ([8.8](08_targyak.md#88-azonosítás-és-nevek)). Megjelenítés az `inven` opció szerint: *overwrite* – a térkép jobb felső sarkára rajzolt keretben, *slow* – soronként az üzenetsorban (`--More--`-val), *clear* – külön teljes képernyőn. A lista végén „--Press space to continue--”; egyetlen sornál üzenetként jelenik meg. Üres csomag: „you are empty handed”.

**Egy tárgy (`I`):** üres csomag: „you aren't carrying anything”; egyetlen tárgy: azonnal „a) <név>”; különben „which item do you wish to inventory: ” → „<x>) <név>” vagy „'<x>' not in pack”.

**Fegyver kézbe (`w`):** ha a jelenlegi fegyver átkozott: „you can't.  It appears to be cursed” (időbe kerül). Páncélra: „you can't wield armor” (nincs idő). Már használt tárgyra: „That's already in use” (nincs idő). Siker: „you are now wielding <név> (<betű>)”.

**Páncél fel (`W`) / le (`T`):** [4.2.3](#423-tárgyak). Siker: „you are now wearing <név>” (a páncél azonosítódik), illetve „you used to be wearing <betű>) <név>”. Nem páncélra: „you can't wear that”.

**Gyűrű fel (`P`):** nem gyűrűre: „it would be difficult to wrap that around a finger” (időbe kerül); használt tárgyra: „That's already in use”; ha mindkét kéz szabad: „left hand or right hand? ” (`l`/`r`; ESC kilép, de a kör elmegy), különben a szabad kézre; két gyűrűnél: „you already have a ring on each hand”. Felhúzáskor: add strength → `chg_str(+mod)`; see invisible → CANSEE (a láthatatlan szörnyek kirajzolódnak); aggravate monster → aggravate ([6.4](06_szornyek.md#64-ébresztés)). Üzenet: „you are now wearing <név> (<betű>)”.

**Gyűrű le (`R`):** nincs gyűrű: „you aren't wearing any rings”; két gyűrűnél kérdezi a kezet. Átkozott: „you can't.  It appears to be cursed”. Levételkor: add strength → `chg_str(−mod)`; see invisible → CANSEE le, és **a see invisible bájital hátralévő hatása is megszűnik**. Üzenet: „was wearing <név>(<betű>)”.

**Elnevezés (`c`):** választható bármi, kivéve ételt („you can't call that anything”). Bájital/tekercs/gyűrű/pálca: a *típus* kap nevet (ha már azonosított: „that has already been identified”); minden más (fegyver, páncél, amulett): az adott tárgy saját címkét kap. Ha már volt név: „Was called "<név>"”; majd „what do you want to call it? ”. A beviteli mező a régi névvel, ha az nincs, az azonosítatlan névvel (szín, felirat, kő, anyag) van előtöltve.

**Felfedezések (`D`):** „for what type of object do you want a list? (* for all)” – `!` `?` `=` `/` vagy `*` (ESC kilép; más billentyűre: „Please type one of !?=/ (ESCAPE to quit)”). Az ismert vagy elnevezett típusok listája minden hívásnál **véletlen sorrendben**, a tárgynév-formátummal ([8.8](08_targyak.md#88-azonosítás-és-nevek)); `*`-nál a négy lista üres sorral elválasztva. Ha nincs semmi: „Haven't discovered anything about any potions” (scrolls / rings / sticks).

**Jel magyarázata (`/`):** „what do you want identified? ” → „'<jel>': <leírás>” (ESC-re az üzenetsor törlődik, nincs válasz). Nagybetű → a szörny neve. Ismert jelek: `|` `-` wall of a room, `*` gold, `%` a staircase, `+` door, `.` room floor, `@` you, `#` passage, `^` trap, `!` potion, `?` scroll, `:` food, `)` weapon, ` ` solid rock, `]` armor, `,` the Amulet of Yendor, `=` ring, `/` wand or staff. Minden más (a `$` is): „unknown character”.

**Súgó (`?`):** „character you want help for (* for all): ” – egy billentyűre az ahhoz tartozó sor („<jel>	<leírás>”, ismeretlenre „unknown character '<x>'”); `*`-ra a teljes lista két oszlopban egy külön képernyőn, „--Press space to continue--”.

Sorformátum: `unctrl(jel)` + leírás, ahol a leírás tabulátorral kezdődik (8-as tabpozíciók; irányos parancsnál a jel után `<dir>` áll a tab előtt; a két „run” sornak nincs jele). A `*` listába 49 sor tartozik. Oszloponként `min(⌈49/2⌉, LINES − 1)` sor fér el, ez 24 soros terminálon 23. A rajzolás a második oszlop végén leáll, így 80×24-en csak 46 sor jelenik meg: a bal oszlop (0. oszloptól) a `?`-tól az `i`-ig, a jobb oszlop (40. oszloptól) az `I`-től a `Q`-ig.
```
?       prints help                     I       inventory single item
/       identify object                 q       quaff potion
h       left                            r       read scroll
j       down                            e       eat food
k       up                              w       wield a weapon
l       right                           W       wear armor
y       up & left                       T       take armor off
u       up & right                      P       put on ring
b       down & left                     R       remove ring
n       down & right                    d       drop object
        <SHIFT><dir>: run that way      c       call object
        <CTRL><dir>: run till adjacent  a       repeat last command
f<dir>  fight till death or near death  )       print current weapon
t<dir>  throw something                 ]       print current armor
m<dir>  move onto without picking up    =       print current rings
z<dir>  zap a wand in a direction       @       print current stats
^<dir>  identify trap type              D       recall what's been discovered
s       search for trap/secret door     o       examine/set options
>       go down a staircase             ^R      redraw screen
<       go up a staircase               ^P      repeat last message
.       rest for a turn                 ^[      cancel command
,       pick something up               S       save game
i       inventory                       Q       quit
```
A lista utolsó három sora (`!` shell escape, `F<dir>` fight till either of you dies, `v` print version number) csak legalább 26 soros terminálon fér ki; egy billentyűs súgóként mindhárom elérhető. Az egyes irányok nagybetűs és CTRL-os változatainak („run left”, „run left until adjacent”, …) saját súgósora is van, de a `*` listában nem szerepelnek.

**Felvétel részletei:** lebegve semmi nem vehető fel (rálépéskor csendben nem történik semmi, a `,` parancsra „You can't.  You're floating off the ground!”). Arany: „you found <n> gold pieces”, és a szoba aranya 0 lesz (a kapzsi szörnyek ettől a játékos felé fordulnak). `m` irány: „you moved onto <név>”. `,` üres mezőn: „there is nothing here to pick up”.

---

## 4.3 Szintváltás és játék vége

```
// mindkét parancs * – nem visz el időt
descend():  // '>'
  if ISLEVIT: "you can't. You're floating off the ground!"; return
  if not on STAIRS: "I see no way down"; return
  level++; new_level()
ascend():   // '<'
  if ISLEVIT: "you can't. You're floating off the ground!"; return
  if not on STAIRS: "I see no way up"; return
  if !has_amulet: "your way is magically blocked"; return
  level--; if level == 0: win()
  else: new_level(); "you feel a wrenching sensation in your gut"
```
**Halál:** a játékos elveszíti az aranya 10%-át (`gold -= gold/10`), pontszám = a maradék arany.
- *Sírkő* (`tombstone` opció be), a képernyő 8. sorától:
```
                       __________
                      /          \
                     /    REST    \
                    /      IN      \
                   /     PEACE      \
                  /                  \
                  |      <név>       |        ← 14. sor, középre igazítva
                  |    <arany> Au    |        ← 15. sor, középre
                  |   killed by a    |        ← 16. sor; „an” magánhangzónál, éhhalál/fagyásnál nincs névelő
                  |    <halálok>     |        ← 17. sor, középre
                  |       <év>       |        ← 18. sor
                 *|     *  *  *      | *
         ________)/\\_//(\/(/\)/\//\/|_)_______
```
- *Sírkő nélkül:* a 22. sorban „Killed by a(n) <halálok> with <arany> gold” (éhhalál/fagyás esetén névelő nélkül).
- Halálokok: szörny → a szörny neve; `arrow`, `bolt`, `dart` (névelővel), `hypothermia` (Ice monster), `starvation` (névelő nélkül).

**Kilépés** (`Q`): pontszám = arany, levonás nélkül.

**Győzelem:** „Congratulations, you have made it to the light of day!” – utána a csomag minden tárgya azonosítódik és eladásra kerül; pontszám = arany + a tárgyak értéke:

| tárgy | érték |
|---|---|
| étel | 2 × darab |
| fegyver | alapérték × (3 × (hplus + dplus) + darab) |
| páncél | alapérték + 100 × (9 − arm) + 10 × (alap védelem − arm) |
| tekercs, bájital | alapérték × darab; ha a típus nem volt ismert, a fele |
| gyűrű | alapérték (+ a kő értéke, [8.8](08_targyak.md#88-azonosítás-és-nevek)); protection / add strength / dexterity / increase damage: + 100 × mod, ha mod > 0, különben összesen 10; ha nem volt azonosítva, a fele |
| pálca/bot | alapérték + 20 × töltés; ha nem volt azonosítva, a fele |
| Amulett | 1000 |

A negatív érték 0-nak számít.

**Ranglista (top 10):** halál, kilépés és győzelem után „[Press return to continue]”, majd:
```
Top Ten Rogueists:
   Score Name
 1  4130 bob: A total winner on level 26.
 2  1210 alice: killed on level 14 by a troll.
 3   302 bob: quit on level 3.
```
- Egy bejegyzés: pontszám, név, ok (`killed`, `quit`, `A total winner`, `killed with Amulet`), szint (győzelemnél a `max_level`, egyébként az aktuális), halálnál a halálok, felhasználóazonosító.
- Beszúrás: a pontszám szerint csökkenő sorrendben, ha nagyobb valamelyik meglévőnél. Felhasználónként csak **egy** nem-győztes bejegyzés maradhat: ha az új jobb, a régi kiesik; ha a felhasználónak már van jobb nem-győztes bejegyzése, az új nem kerül fel. Győzelmek korlátlanul.
- Az új bejegyzés kiemelve (inverz) jelenik meg. A fájl írása zárolással történik.
