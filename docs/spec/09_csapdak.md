# 9. Csapdák

## 9.1 Rálépés
```
trigger_trap(P):
  if player.ISLEVIT: return                    // lebegve a rejtett csapda rejtve is marad
  MAP[P] &= ~HIDDEN; SCREEN[P] = TRAP
  futás/ismétlés megszakad
  switch type:
    TRAPDOOR: level++; new_level() (max_level frissül); "you fell into a trap!"
    ARROW:    7.5
    SLEEP:    no_command += 5; ISRUN le; "a strange white mist envelops you and you fall asleep"
    BEAR:     bear_trap += 3; "you are caught in a bear trap"
    TELEPORT: teleport()   (a csapda jele a régi helyén megmarad)
    DART:     7.5
    RUST:     "a gush of water hits you on the head"; rust_armor()  (6.7)
    MYSTERY:  rnd(11) szerint egy üzenet, hatás nincs ({szín} = véletlen bájitalszín, 8.8):
      "you are suddenly in a parallel dimension"
      "the light in here suddenly seems {szín}"
      "you feel a sting in the side of your neck"
      "multi-colored lines swirl around you, then fade"
      "a {szín} light flashes in your eyes"
      "a spike shoots past your ear!"
      "{szín} sparks dance across your armor"
      "you suddenly feel very thirsty"
      "you feel time speed up suddenly"
      "time now seems to be going slower"
      "you pack turns {szín}!"
  billentyűpuffer ürítése
```
Rálépéskor a játék nem ír csapdanevet, így hallucináció alatt sem cserél nevet; a véletlen név a `^` paranccsal és a keresés („you found …”) üzenetében jelenik meg.
A csapdák állandók, minden rálépéskor újra aktiválódnak.

## 9.2 Keresés (`s`, searching gyűrű)
```
search():
  for each of 8 neighbours N with HIDDEN:
    k = (ISHALU ? 3 : 0) + (ISBLIND ? 2 : 0)
    DOOR:    if rnd(5 + k) == 0: reveal
    TRAP:    if rnd(2 + k) == 0: reveal, "you found <trap name>"
    PASSAGE: if rnd(3 + k) == 0: reveal
  reveal: HIDDEN bit le, SCREEN frissül, futás/ismétlés megszakad
```
