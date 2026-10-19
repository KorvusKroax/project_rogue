# 3. Látótér és megjelenítés

## 3.1 Felderítés
```
enter_room(P):                      // ajtón át belépéskor, teleportkor, új szinten
  proom = a P-t tartalmazó szoba; door_open(proom) (ébresztés, 6.4)
  if !DARK(proom) and !ISBLIND:
     a teljes szoba (falakkal) MAP → SCREEN, a szörnyek helyén a látható szörnyek (see_monst)

leave_room(P):                      // szobából ajtón át folyosóra lépéskor
  if MAZE(proom): return
  floor = PHANTOM ? PASSAGE : (!DARK or ISBLIND) ? FLOOR : ' '
  a szoba SCREEN-jén: sötét szobában a FLOOR jelek ' '-re törlődnek;
  a szörnyek jele helyére floor (ajtón DOOR) kerül – SEEMONST alatt inverzen maradnak
  proom = a folyosórendszer; door_open(régi szoba)

look(wakeup):                       // minden akció előtt
  ha elmozdult: erase_lamp(régi pozíció, régi szoba)
  a 3×3 környezet minden mezője (a játékosé kivételével):
     ' ' mező kimarad; folyosó és szoba közt csak ajtón át lát (F_PASS-egyezés);
     sarkon át átlósan nem lát
     szörny: ha SEEMONST és a szörny ISINVIS: a mező teljesen kimarad (nem rajzol, nem ébreszt;
             CTRL-futásnál a futás megáll)
             különben wakeup esetén wake_monster; ha see_monst → a jele (hallucinálva véletlen betű),
             ha nem látszik → a MAP karaktere (trip_ch nélkül)
     egyébként a MAP karaktere, trip_ch-n át (hallucinálva véletlen tárgyjel, de csak ha az akció időt vitt)
     sötét szobában, seefloor ki: a FLOOR helyett ' '
  vakon csak a játékos mezőjét rajzolja, de a szomszédos szörnyek ébresztése vakon is lefut

erase_lamp(pos, room):              // csak sötét, nem fantom szobában, seefloor be, nem vak
  a pos körüli 3×3-ban a SCREEN FLOOR jelei ' '-re (a játékos mezője kivételével)
```
- **Ami a SCREEN-en marad (memória):** falak, ajtók, folyosók, lépcső, felfedezett csapdák **és a már látott tárgyak**. Sötét szobából távozva *kizárólag a FLOOR karakterek* törlődnek.
- **Tárgyak:** a látott tárgy emlékként a helyén marad akkor is, ha már nincs fény/rálátás. Az emlék nem frissül: ha közben egy szörny felveszi vagy elmozdítja, a képernyőn a régi jel marad, amíg a játékos újra nem látja a mezőt. Sötét szobában az új tárgy csak akkor jelenik meg, ha a 3×3-as környezetbe kerül; világos szobában belépéskor az összes látszik.
- **Szörnyek:** sosem maradnak emlékként. Csak akkor látszanak, ha *éppen* láthatók: világos szobában a játékossal azonos szobában, sötét szobában és folyosón csak a 3×3-ban (dist2 < 3), egyébként SEEMONST/CANSEE szabályai szerint. Ha nem látható, a helyén az alatta lévő emlékezett mező (`oldch`) jelenik meg.
- Rejtett (F_REAL nélküli) mezők a rejtett alakjukban jelennek meg, amíg fel nem fedezik őket.

## 3.2 Állapotok hatása a kijelzésre
- **ISBLIND:** a `look` csak a játékos saját mezőjét frissíti, szoba nem világosodik ki, szörny nem látszik (`see_monst` hamis). Visszanyeréskor (`sight`): ha a játékos szobában van, `enter_room`; „the veil of darkness lifts” (hallucinálva: „far out!  Everything is all cosmic again”).
- **ISHALU** – két része van:
  - `look` közben (`trip_ch`): minden látott mező, ami nem padló, üres, folyosó, fal, ajtó vagy csapda – és nem a már látott lépcső –, véletlen tárgyjelként (`rnd_thing`, [6.2](06_szornyek.md#62-generálás)) rajzolódik; a látható szörnyek véletlen nagybetűként.
  - minden kör elején (`visuals` démon, BEFORE fázis; futás közben `jump` mellett kimarad): a látható tárgyak, a még nem látott lépcső és a látható szörnyek újrasorsolódnak (az álcázott Xeroc tárgyjelet kap); SEEMONST mellett a falon túli szörnyek is véletlen betűt kapnak.
  - Lejáratkor (`come_down`): a látható tárgyak és szörnyek a valódi jelükkel rajzolódnak vissza; „Everything looks SO boring now.” Ha a játékos épp vak, csak a flag törlődik: nincs visszarajzolás és üzenet sem.
- **SEEMONST** (monster detection): bekapcsoláskor minden szörny a valódi típusjelével (hallucinálva véletlen betűvel) rajzolódik, a nem látottak inverzen; lejártakor a nem látott szörnyek helyére az `oldch` kerül. Ha egyetlen nem látott szörny sincs: „you have a strange feeling for a moment, then it passes” (hallucinálva „normal”).
- **CANSEE:** bekapcsoláskor (`invis_on`) a látható ISINVIS szörnyek kirajzolódnak (hallucinálva nem); kikapcsoláskor (`unsee`) helyükre az `oldch` kerül.
- **Mágia-észlelés:** külön ablakban a szint és a szörnyek csomagjainak mágikus tárgyai `$` jellel; „You sense the presence of magic on this level.--More--”, billentyűre visszaáll; nem telik idő.
- **Étel-észlelés:** ugyanígy, az ételek `:` jellel ([8.3](08_targyak.md#83-tekercsek-r)).
- **Térképezés:** a MAP adott részei közvetlenül a SCREEN-re ([8.3](08_targyak.md#83-tekercsek-r)).
- A hallucinált üzenetváltozatok (`choose_str`): több üzenetnek van „trippy” párja (pl. „you feel less trippy now”, „the munchies are interfering with your motor capabilites”); ezeket az adott üzenetnél zárójelben jelöli a specifikáció, ahol eltérnek.

## 3.3 Státuszsor
A 23. sorban:
```
printf("Level: %d  Gold: %-5d  Hp: %*d(%*d)  Str: %2d(%d)  Arm: %-2d  Exp: %d/%d  %s",
       level, gold, hpw, hpt, hpw, maxhp, str, str_max, 10 - armor, lvl, exp, hunger_word)
// hpw = a maxhp számjegyeinek száma (igazítás); hunger_word: "", "Hungry", "Weak", "Faint"
// armor = viselt páncél arm-ja (vagy a játékos alap arm-ja, 10)
//   a védelem-gyűrű a kijelzett értékbe NEM számít bele (a harcban igen, 5.5)
pl.  "Level: 3  Gold: 25     Hp: 14(20)  Str: 16(16)  Arm: 5   Exp: 2/38  Hungry"
```
Csak akkor rajzolódik újra, ha valamelyik érték változott. A `@` parancs ugyanezt az üzenetsorba írja.

## 3.4 Üzenetsor
- A 0. sorban; minden üzenet nagybetűvel kezdődik (kivéve a csomagbetűvel kezdődőket, pl. „a) …”).
- Az egy körben keletkező üzenetek összefűződnek. Ha az eddigi szöveg és az új rész együttes hossza eléri a 71 karaktert (`MAXMSG = 80 − sizeof("--More--")`, a sizeof a záró nullát is számolja), az eddigi sor után `--More--` jelenik meg, és a játék szóközre vár.
- A következő billentyűleütéskor az üzenetsor törlődik. `CTRL-P` az utolsó üzenetet ismétli.
