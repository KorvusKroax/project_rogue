# Függelék – üzenetkatalógus

A játék összes üzenete, a megjelenés helyzetével. Szabályok ([3.4](03_latoter_es_megjelenites.md#34-üzenetsor)):
- Az üzenetsorba írt szöveg első betűje **nagybetűre** változik (kivéve, ha csomagbetűvel kezdődik, pl. „a) …”, vagy a súgósort). Az alábbi táblákban a forrásbeli kisbetűs alak szerepel.
- A mondatok közötti **két szóköz** (pl. „you can't.  It appears…”) az eredeti szöveg része.
- `<…>` = behelyettesített érték; `<szín>` hallucináció alatt véletlen bájitalszín ([8.8](08_targyak.md#88-azonosítás-és-nevek)), egyébként a megadott szín (`pick_color`).
- **terse** oszlop: ha üres, terse módban is ugyanaz. **hallucinálva** oszlop: ha üres, nincs külön változat.

## Parancsciklus, általános
| helyzet | normál | terse | hallucinálva |
|---|---|---|---|
| alvás/ájulás/fagyás vége | you can move again | | |
| ismeretlen billentyű | illegal command '<bill>' | | |
| `a` első parancsként | you haven't typed a command yet | | |
| `v` | version 5.4.4. (mctesq was here) | | |
| irány bekérése | which direction? | direction: | |
| `f` olyan irányba, ahol nincs látott szörny | I see no monster there | no monster there | |
| foglalt tárgy újrahasználata | That's already in use | in use | |
| átkozott tárgy levétele | you can't.  It appears to be cursed | | |
| `)` | you are wielding (<b>) <név> / you are wielding nothing | <b>) <név> / wielding nothing | |
| `]` | you are wearing (<b>) <név> / you are wearing nothing | <b>) <név> / wearing nothing | |
| `=` (kezenként külön üzenet) | you are wearing (<b>) <név> on left hand / you are wearing nothing on right hand | … (L) / … (R) | |
| `@` | a státuszsor szövege ([3.3](03_latoter_es_megjelenites.md#33-státuszsor)) | | |

## Mozgás, lépcső
| helyzet | normál | terse | hallucinálva |
|---|---|---|---|
| medvecsapdában lépne | you are still stuck in the bear trap | | |
| flytrap fogja | you are being held | | |
| `m` tárgyra | you moved onto <név> | moved onto <név> | |
| `,` üres mezőn | there is nothing here to pick up | nothing here | |
| lebegve `,` `>` `<` | You can't.  You're floating off the ground! | | |
| `>` nem lépcsőn | I see no way down | | |
| `<` nem lépcsőn | I see no way up | | |
| `<` amulett nélkül | your way is magically blocked | | |
| `<` sikeres | you feel a wrenching sensation in your gut | | |
| szintlépés | welcome to level <n> | | |

## Tárgyak kezelése
| helyzet | normál | terse | hallucinálva |
|---|---|---|---|
| tárgy bekérése | which object do you want to <cél>? (* for list): | <cél> what? (* for list): | |
| üres csomag | you aren't carrying anything | | |
| `a` ismétlés, elfogyott | you ran out | | |
| rossz betű | '<b>' is not a valid item | | |
| `*` lista, nincs megfelelő | you don't have anything appropriate | nothing appropriate | |
| `i` üres csomag | you are empty handed | empty handed | |
| lista vége | --Press space to continue-- | | |
| `I` kérdés | which item do you wish to inventory: | item: | |
| `I` rossz betű | '<b>' not in pack | | |
| felvétel | you now have <név> (<b>) | <név> (<b>) | |
| tele csomag | there's no room in your pack | no room | |
| arany | you found <n> gold pieces | <n> gold pieces | |
| scare monster másodszor | the scroll turns to dust as you pick it up | | |
| lerakás foglalt mezőre | there is something there already | | |
| lerakás | dropped <név> | | |
| dobott tárgy nem fér le | the <fegyvernév> vanishes as it hits the ground | | |
| `w` páncélra | you can't wield armor | | |
| `w` siker | you are now wielding <név> (<b>) | wielding <név> (<b>) | |
| `W` nem páncél | you can't wear that | | |
| `W` már van rajta | you are already wearing some.  You'll have to take it off first | you are already wearing some | |
| `W` siker | you are now wearing <név> | wearing <név> | |
| `T` nincs páncél | you aren't wearing any armor | not wearing armor | |
| `T` siker | you used to be wearing <b>) <név> | was wearing <b>) <név> | |
| `P` nem gyűrű | it would be difficult to wrap that around a finger | not a ring | |
| `P` két gyűrű van | you already have a ring on each hand | wearing two | |
| kéz kérdése | left hand or right hand? | left or right ring? | |
| kéz – rossz válasz | please type L or R | L or R | |
| `P` siker | you are now wearing <név> (<b>) | <név> (<b>) | |
| `R` nincs gyűrű | you aren't wearing any rings | no rings | |
| `R` üres kéz | not wearing such a ring | | |
| `R` siker | was wearing <név>(<b>) | | |
| `c` ételre | you can't call that anything | | |
| `c` azonosítottra | that has already been identified | | |
| `c` régi név | Was called "<név>" | called "<név>" | |
| `c` kérdés (és ismeretlen tárgy használata után) | what do you want to call it? | call it: | |
| `D` kérdés | for what type of object do you want a list? (* for all) | what type? (* for all) | |
| `D` rossz válasz | Please type one of !?=/ (ESCAPE to quit) | Not a type | |
| `D` semmi | Haven't discovered anything about any <potions/scrolls/rings/sticks> | Nothing about any … | |
| `/` kérdés | what do you want identified? | | |
| `/` válasz | '<jel>': <leírás> (leírások: [4.2.6](04_jatekciklus_es_parancsok.md#426-tárgyválasztás-listák-és-tárgyparancsok)) | | |
| `?` kérdés | character you want help for (* for all): | | |
| `?` ismeretlen | unknown character '<jel>' | | |
| `e` nem ételre | ugh, you would get ill if you ate that | that's Inedible! | |
| evés: gyümölcs | my, that was a yummy <gyümölcs> | | |
| evés: rossz ízű (+1 XP) | yuk, this food tastes awful | | bummer, this food tastes awful |
| evés: jó | yum, that tasted good | | oh, wow, that tasted good |
| `q` nem bájitalra | yuk! Why would you want to drink that? | that's undrinkable | |
| `r` nem tekercsre | there is nothing on it to read | nothing to read | |
| `z` nem pálcára | you can't zap with that! | | |
| azonosító tekercs: üres csomag | you don't have anything in your pack to identify | | |
| azonosító tekercs: nem választott | you must identify something | | |
| azonosító tekercs: rossz típus | you must identify a <potion/scroll/weapon/armor/ring, wand or staff> | | |

## Bájitalok
| bájital | normál | hallucinálva |
|---|---|---|
| confusion | wait, what's going on here. Huh? What? Who? | what a tripy feeling! |
| hallucination | Oh, wow!  Everything seems so cosmic! | (ugyanaz) |
| poison | you feel very sick now (sustain strength gyűrűvel: you feel momentarily sick) | |
| gain strength | you feel stronger, now.  What bulging muscles! | |
| see invisible | this potion tastes like <gyümölcs> juice | |
| healing | you begin to feel better | |
| monster detection, ha nincs mit mutatni | you have a strange feeling for a moment, then it passes | you have a normal feeling … |
| magic detection, ha van | You sense the presence of magic on this level.--More-- (külön képernyőn) | |
| magic detection, ha nincs | you have a strange feeling for a moment, then it passes | you have a normal feeling … |
| raise level | you suddenly feel much more skillful (+ welcome to level <n>) | |
| extra healing | you begin to feel much better | |
| haste self | you feel yourself moving much faster | |
| haste self, már gyors | you faint from exhaustion | |
| restore strength | hey, this tastes great.  It make you feel warm all over | |
| blindness | a cloak of darkness falls around you | oh, bummer!  Everything is dark!  Help! |
| levitation | you start to float in the air | oh, wow!  You're floating in the air! |

## Tekercsek
| tekercs / helyzet | normál | terse | hallucinálva |
|---|---|---|---|
| monster confusion | your hands begin to glow <red> | | |
| magic mapping | oh, now this scroll has a map on it | | |
| hold monster – 1 szörny | the monster freezes | | |
| hold monster – több | the monsters around you freeze | | |
| hold monster – nincs | you feel a strange sense of loss | | |
| sleep | you fall asleep | | |
| enchant armor | your armor glows <silver> for a moment | | |
| identify … | this scroll is an <identify potion / …> scroll | | |
| scare monster | you hear maniacal laughter in the distance | | |
| food detection – van | Your nose tingles and you smell food.--More-- (külön képernyőn) | | |
| food detection – nincs | your nose tingles | | |
| enchant weapon | your <fegyvernév> glows <blue> for a moment | | |
| enchant weapon – nincs fegyver | you feel a strange sense of loss | | |
| create monster – nincs hely | you hear a faint cry of anguish in the distance | | |
| remove curse | you feel as if somebody is watching over you | | you feel in touch with the Universal Onenes |
| aggravate monsters | you hear a high pitched humming noise | | |
| protect armor | your armor is covered by a shimmering <gold> shield | | |
| protect armor – nincs páncél | you feel a strange sense of loss | | |

## Pálcák, botok, sugarak
| helyzet | normál | terse |
|---|---|---|
| üres pálca | nothing happens | |
| light – folyosón | the corridor glows and then fades | |
| light – szobában | the room is lit by a shimmering <blue> light | the room is lit |
| drain life – túl gyenge | you are too weak to use it | |
| drain life – nincs célpont | you have a tingling feeling | |
| magic missile – mellé | the missle vanishes with a puff of smoke | missle vanishes |
| sugár visszapattan | the <bolt/flame/ice> bounces | |
| láng a sárkányon | the flame bounces off the dragon | the flame bounces |
| sugár elkerüli a szörnyet | the <bolt> whizzes past <szörny> | <bolt> misses |
| sugár eltalálja a játékost | you are hit by the <bolt> | the <bolt> hits |
| sugár elkerüli a játékost | the <bolt> whizzes by you | |

## Harc és szörnyek
| helyzet | normál | terse | hallucinálva |
|---|---|---|---|
| ütés/mellé ([7.7](07_harc.md#77-harci-üzenetek)) | <Támadó> + h_names/m_names + <célpont> | You hit / You miss / <Szörny> hit / misses | |
| dobott tárgy talál | the <fegyver> hits <szörny> (nem fegyver: you hit <szörny>) | | |
| dobott tárgy mellé | the <fegyver> misses <szörny> (nem fegyver: you missed <szörny>) | | |
| xeroc leleplezése | wait!  That's a xeroc! | | heavy!  That's a nasty critter! |
| monster confusion elsül | your hands stop glowing <red> | | |
| szörny összezavarodik | <szörny> appears confused | | |
| szörny meghal | you have defeated <szörny> (előző találat után: .  Defeated <szörny>) | defeated <szörny> | |
| Ice monster | you are frozen by the <szörny> ([7.7](07_harc.md#77-harci-üzenetek)) | you are frozen | |
| Rattlesnake – erővesztés | you feel a bite in your leg and now feel weaker | a bite has weakened you | |
| Rattlesnake – sustain gyűrű | a bite momentarily weakens you | bite has no effect | |
| Vampire / Wraith szívás | you suddenly feel weaker | | |
| Leprechaun | your purse feels lighter | | |
| Nymph | she stole <név>! | | |
| Medusa pillantása | <szörny>'s gaze has confused you (pl. the medusa's gaze …; ha nem látható: its gaze has confused you) | | |
| rozsda (Aquator, csapda) | your armor appears to be weaker now. Oh my! | your armor weakens | |
| rozsda – védett páncél | the rust vanishes instantly | | |
| szörnynév, ha nem látható | something | it | |

## Csapdák és keresés
| helyzet | normál | terse |
|---|---|---|
| csapóajtó | you fell into a trap! | |
| medvecsapda | you are caught in a bear trap | |
| altatógáz | a strange white mist envelops you and you fall asleep | |
| nyíl talál | oh no! An arrow shot you (halálkor: an arrow killed you) | |
| nyíl mellé | an arrow shoots past you | |
| dart talál | a small dart just hit you in the shoulder (halálkor: a poisoned dart killed you) | |
| dart mellé | a small dart whizzes by your ear and vanishes | |
| rozsdacsapda | a gush of water hits you on the head | |
| rejtélyes csapda | a 11 üzenet egyike ([9.1](09_csapdak.md#91-rálépés)) | |
| `^` – van csapda | You have found <csapdanév> | <csapdanév> |
| `^` – nincs/rejtett | You have found no trap there | no trap there |
| keresés – titkos ajtó | a secret door | |
| keresés – csapda | you found <csapdanév> | <csapdanév> |

Csapdanevek (hallucinálva véletlen): a trapdoor, an arrow trap, a sleeping gas trap, a beartrap, a teleport trap, a poison dart trap, a rust trap, a mysterious trap.

## Éhség és lejáró hatások
| helyzet | normál | terse | hallucinálva |
|---|---|---|---|
| éhes lesz | you are starting to get hungry | getting hungry | you are getting the munchies (terse: getting the munchies) |
| gyenge lesz | you are starting to feel weak | | the munchies are interfering with your motor capabilites |
| elájul | you feel too weak from lack of food.  You faint | You faint | the munchies overpower your motor capabilities.  You freak out |
| confusion vége | you feel less confused now | | you feel less trippy now |
| vakság vége | the veil of darkness lifts | | far out!  Everything is all cosmic again |
| hallucináció vége | Everything looks SO boring now. | | |
| gyorsítás vége | you feel yourself slowing down | | |
| lebegés vége | you float gently to the ground | | bummer!  You've hit the ground |

## Mentés, kilépés, indítás, vége
| helyzet | szöveg |
|---|---|
| indítás | Hello <név>, just a moment while I dig the dungeon... |
| kis terminál | Sorry, the screen must be at least 24x80 |
| mentés – ismert fájlnév | save file (<fájl>)? |
| mentés – rossz válasz | please answer Y or N |
| mentés – fájlnév | file name: |
| mentés – létező fájl | File exists.  Do you wish to overwrite it? (rossz válaszra: Please answer Y or N) |
| betöltés után | file name: <fájl> |
| betöltés – régi verzió | Sorry, saved game is out of date. |
| betöltés – nagyobb képernyő | Sorry, original game was played on a screen with <n> lines. / Current screen only has <n> lines. Unable to restore game (oszlopokra ugyanígy: columns) |
| betöltés – linkelt fájl | Cannot restore from a linked file |
| betöltés – törlés sikertelen | Cannot unlink file |
| betöltés – halott játékos | "He's dead, Jim" |
| kilépés kérdése | really quit? |
| kilépés | You quit with <n> gold pieces |
| halál, sírkő nélkül | Killed by a(n) <ok> with <n> gold |
| halál/kilépés/győzelem után | [Press return to continue] |
| ranglista fejléc | Top Ten Rogueists: / Score Name |
| ranglista – foglalt fájl | The score file is very busy.  Do you want to wait longer / for it to become free so your score can get posted? / If so, type "y" |
| győzelem | inverz kiemeléssel a lenti „Yay, you made it!”-szerű @-rajz, majd: Congratulations, you have made it to the light of day! / You have joined the elite ranks of those who have escaped the / Dungeons of Doom alive.  You journey home and sell all your loot at / a great profit and are admitted to the Fighters' Guild. / --Press space to continue-- |
| győzelem – eladás | Worth  Item, soronként „<b>) <érték>  <név>”, végül „<n>  Gold Pieces” |

Győzelmi felirat (a képernyő tetején, inverzen, utána a „Congratulations…” sor is inverz):
```

  @   @               @   @           @          @@@  @     @
  @   @               @@ @@           @           @   @     @
  @   @  @@@  @   @   @ @ @  @@@   @@@@  @@@      @  @@@    @
   @@@@ @   @ @   @   @   @     @ @   @ @   @     @   @     @
      @ @   @ @   @   @   @  @@@@ @   @ @@@@@     @   @     @
  @   @ @   @ @  @@   @   @ @   @ @   @ @         @   @  @
   @@@   @@@   @@ @   @   @  @@@@  @@@@  @@@     @@@   @@   @

     Congratulations, you have made it to the light of day!
```
A ranglista fejléce „Top Ten Rogueists:”; ha a játék „minden pontszámot” módban van fordítva (felhasználónkénti korlát nélkül), „Top Ten Scores:”.

## Szövegtáblák máshol
- Harci igék: [7.7](07_harc.md#77-harci-üzenetek) (`h_names`, `m_names`).
- Rejtélyes csapda üzenetei: [9.1](09_csapdak.md#91-rálépés).
- Súgó sorai és a `/` jelleírásai: [4.2.6](04_jatekciklus_es_parancsok.md#426-tárgyválasztás-listák-és-tárgyparancsok).
- Tárgynevek, színek, kövek, anyagok, tekercsfeliratok: [8.8](08_targyak.md#88-azonosítás-és-nevek).
- Halálokok a sírkövön és a ranglistán: 14.
- Az opciók menüje (`o`): soronként „<leírás> ("<név>"): <érték>” (logikai: True/False, bekéréskor „(T or F)”; inven: Overwrite/Slow/Clear, bekéréskor „(O, S, or C)”), a végén „--Press space to continue--”.

*Nem része a játéknak:* a varázsló mód üzenetei, a belső hibaüzenetek („in some bizarre place (…)”, „warning, connectivity problem on this level”), valamint a rendszerterhelést figyelő fordítási változatok (MAXLOAD, MAXUSERS, CHECKTIME) üzenetei (pl. „Sorry, <név>, but the system is too loaded now.” / „Try again later.  Meanwhile, why not enjoy a <gyümölcs>?”).
