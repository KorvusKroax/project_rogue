# 0. Áttekintés

A *Rogue: Exploring the Dungeons of Doom* 5.4.4-es változatának nyelvfüggetlen specifikációja: a játék minden szabálya, képlete, táblázata és üzenete, amely alapján a program megírható.

**Futási környezet:** 80×24-es karakteres terminál (curses), teljes billentyűzet SHIFT/CTRL kombinációkkal, fájlrendszer a mentéshez és a ranglistához.

**Jelölések:** a pszeudokód C-szerű; „MUST”-szintű szabály minden, ami le van írva – a szokatlannak tűnő viselkedések is (ezeket *Megjegyzés* jelöli). A játék adattáblái annál a fejezetnél vannak, amelyik a szabályaikat leírja; az összes üzenetét a [függelék](fuggelek_uzenetkatalogus.md) tartalmazza. A specifikációt az eredeti 5.4.4 C forráskóddal fejezetenként összevetették, ezért a megvalósításhoz a forrásra nincs szükség.

A pseudokód C-szerű, egész aritmetikával (osztás lefelé kerekít).

- Körökre osztott, egyképernyős, ASCII karakteres roguelike.
- A játékos az 1. dungeon szinten kezd. A cél: megszerezni az **Amulettet** (a 26. szinten és mélyebben jelenik meg), majd felmenni vele a lépcsőkön a felszínre. Győzelem: az 1. szinten `<` az Amulettel (`level` 0-ra csökken).
- Minden szint új, véletlenszerűen generált; a korábbi szintek nem tárolódnak (lépcsőn bármelyik irányba mindig új szint generálódik).
- Halál: HP ≤ 0, éhhalál, fagyás (Ice monster), szint/maxhp-elszívás.

## 0.1 Indítás
```
rogue                 új játék
rogue <fájl>          mentett játék betöltése (4.2.5); "rogue -r" a beállított mentésfájlt tölti
rogue -s              csak a ranglista kiírása (4.3)
rogue -d              teszt-halál: véletlen arany (1..100), szint (1..100) és halálok, sírkő + ranglista

indulás:
  ROGUEOPTS környezeti változó feldolgozása (4.2.5); name alapértéke a felhasználónév
  seed beállítása (0.2); ranglista-fájl megnyitása
  "Hello <név>, just a moment while I dig the dungeon..."
  init: valószínűségi táblák, játékos (5.1), tekercsnevek, bájitalszínek, gyűrűkövek, pálcaanyagok (8.8)
  ha a terminál kisebb 80×24-nél: "Sorry, the screen must be at least 24x80", kilépés
  new_level() (2)
  démonok indítása: szörnymozgás, regeneráció, éhség (AFTER); vándorló-szörny időzítő spread(70) (AFTER)
  főciklus (4.1)
```
A forrásban lévő „varázsló” (wizard) mód egy külön fordítási változat hibakereső eszköze, a játék része nem.

---

## 0.2 Alapprimitívek

```
RN              -> seed = seed * 11109 + 13849;  return (seed >> 16) & 0xFFFF     // 32 bites előjeles egész aritmetika
rnd(n)          -> n == 0 ? 0 : abs(RN) % n          // 0..|n|-1
seed            -> induláskor: dnum = (idő másodpercben) + folyamatazonosító; seed = dnum
chance(p%)      -> rnd(100) < p
ROLL(n, s)      -> n darab s oldalú kocka összege: sum(i<n) (rnd(s) + 1)
spread(n)       -> n - n/20 + rnd(n/10)          // kb. n * 0.95..1.05
dist2(a, b)     -> (a.x-b.x)^2 + (a.y-b.y)^2
sign(v)         -> -1, 0, 1
```

**Sebzés-string** (`"NxM"` vagy `"NxM/NxM/..."`): `/` mentén szegmensekre bontva, mindegyik szegmens egy önálló támadás `ROLL(N, M)` sebzéssel. `"0x0"` = nulla sebzés (a találat ettől még számít, pl. speciális hatásnál).

**Irányok** (8 irány, vi-billentyűk):

| bill. | dx,dy | bill. | dx,dy |
|---|---|---|---|
| h | -1, 0 | y | -1,-1 |
| j | 0, +1 | u | +1,-1 |
| k | 0, -1 | b | -1,+1 |
| l | +1, 0 | n | +1,+1 |
