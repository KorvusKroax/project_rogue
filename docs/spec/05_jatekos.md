# 5. Játékos

## 5.1 Kezdés
```
init_player():
  str = 16; str_max = 16; exp = 0; lvl = 1; arm = 10; dmg = "1x4"; hpt = maxhp = 12
  food_left = HUNGERTIME (1300)
  pack (ebben a sorrendben kerülnek a csomagba, így ezek a betűik):
    a) 1 adag étel (food ration)
    b) Ring mail:  arm = 7 - 1 = 6, ISKNOW, viselve
    c) Mace:       hplus = +1, dplus = +1, ISKNOW, kézben
    d) Short bow:  hplus = +1, dplus = 0, ISKNOW
    e) Arrows:     count = rnd(15) + 25 (25..39), ISKNOW, ISMANY
```

## 5.2 Szintlépés
```
add_exp(amount):
  exp += amount
  new_lvl = 1 + count(xp_next[i] <= exp)  // hány küszöböt ért el
  if new_lvl > lvl:
     hp_gain = ROLL(new_lvl - lvl, 10)    // szintenként 1d10 HP
     maxhp += hp_gain; hpt += hp_gain
     lvl = new_lvl; msg "welcome to level %d"
```

```
xp_next[] = { 10, 20, 40, 80, 160, 320, 640, 1300, 2600, 5200, 13000, 26000,
              50000, 100000, 200000, 400000, 800000, 2000000, 4000000, 8000000 }
// szint = 1 + (hány küszöböt ért el az exp); max. 21. szint
```

Szintcsökkenés (Wraith): `lvl -= 1; exp = xp_next[lvl-1] + 1` az új `lvl`-lel (1. szinten exp=0 → halál, lásd [6.7](06_szornyek.md#67-speciális-támadások-sikeres-találat-után-ha-nincs-iscanc)).

## 5.3 HP regeneráció
```
player_regen():
  regen_counter++
  if lvl < 8:
     if regen_counter + 2*lvl > 20: hpt += 1; regen_counter = 0
  else:
     if regen_counter >= 3: hpt += rnd(lvl-7)+1; regen_counter = 0
  for each hand with Regeneration ring: hpt += 1; regen_counter = 0
  hpt = min(hpt, maxhp)
// regen_counter = 0 harcnál és minden HP-változásnál
```

## 5.4 Éhség
```
hunger():  // minden körben, alvás/ájulás alatt is
  old_state = hungry_state
  if food_left <= 0:
     if food_left-- < -850: death("starvation")  // ≤0 alatt fix 1/kör fogy, a gyűrűk és az amulett nem számít
     if no_command == 0 and rnd(5) == 0:         // 20% ájulás, ha épp nem ájult
        no_command += rnd(8) + 4; hungry_state = 3 (FAINT)
        "you feel too weak from lack of food.  You faint"
  else:
     food_before = food_left
     food_left -= ring_eat(LEFT) + ring_eat(RIGHT) + 1 - (has_amulet ? 1 : 0)
     if food_left < 150 and food_before >= 150: hungry_state = 2 (WEAK);   "you are starting to feel weak"
     else if food_left < 300 and food_before >= 300: hungry_state = 1 (HUNGRY); "you are starting to get hungry"
  if hungry_state != old_state:
     player.ISRUN = false                        // lásd lent
     futás, ismétlés, 'f'-harc megszakad

ring_eat(hand):  // a kézen lévő gyűrű étel/kör értéke (8.6 tábla) szerint; üres kéz: 0
  +k → k;  −k → (rnd(k)==0 ? 1 : 0);  0 → 0;  slow digestion → (rnd(2)==0 ? -1 : 0)
```
- A „Faint” állapot csak az első ájuláskor kapcsol be, addig `food_left ≤ 0` mellett is „Weak” látszik.
- **A player ISRUN jelentése:** a játékos cselekvőképességét a `no_command` számláló kezeli, az ISRUN-nak egyetlen hatása, hogy **ISRUN nélküli védekezőre a támadó +4 találati bónuszt kap** ([7.1](07_harc.md#71-közelharci-találat)). *Megjegyzés:* éhség-állapotváltás után ezért a szörnyek +4-gyel találnak, amíg egy alvás/ájulás/fagyás le nem jár (`"you can move again"`).

```
eat():
  if food_left < 0: food_left = 0
  food_left = min(food_left + rnd(400) + 1100, STOMACH_SIZE)  // STOMACH_SIZE = 2000
  hungry_state = OK
  if food.which == FRUIT: "my, that was a yummy <fruit>"      // food: a megevett tárgy
  else if rnd(100) > 70: exp++; "yuk, this food tastes awful"; szintlépés-ellenőrzés
  else: "yum, that tasted good"
```
A gyümölcs nem evéskor dől el: az étel tárgy generálásakor 10% eséllyel gyümölcs (`which = 1`), 90% food ration ([8.1](08_targyak.md#81-generálás)). A gyümölcs alapneve „slime-mold” (a `fruit` opcióval átnevezhető).

## 5.5 Effektív páncél (védekezéskor)
```
player_armor():
  armor_value = cur_armor ? cur_armor.arm : 10
  for each Protection ring: armor_value -= ring.arm
  return armor_value
// a státuszsor NEM ezt mutatja: ott a gyűrűk nélküli érték látszik (3.3)
```

## 5.6 Erő
```
// az add strength gyűrű bónusza közvetlenül a str értékébe épül be (felhúzáskor +mod, levételkor −mod)
add_str(strength, amount): strength += amount; strength = clamp(strength, 3, 31)
chg_str(amount):
  if amount == 0: return
  add_str(str, amount)
  base_str = str − (a viselt add strength gyűrűk mod-jai, add_str-rel)  // az erő gyűrűk nélkül
  if base_str > str_max: str_max = base_str                             // a maximum a gyűrűk nélküli erő
restore strength (bájital): gyűrűbónuszok le; if str < str_max: str = str_max; gyűrűbónuszok vissza
```
A harci táblák (`str_plus`, `add_dam`) a `str` aktuális értékével indexelnek (index = erő, 0..31):
```
str_plus[] = { -7,-6,-5,-4,-3,-2,-1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 3 }
add_dam[]  = { -7,-6,-5,-4,-3,-2,-1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 2, 3, 3, 4, 5, 5, 5, 5, 5, 5, 5, 5, 5, 6 }
```
