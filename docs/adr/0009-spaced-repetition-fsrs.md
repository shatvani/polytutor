# ADR-0009: Ismétlés-ütemezés – FSRS

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
Szavak és nyelvtani pontok ismétlését ütemezni kell. Több vizsgált alkalmazás (Praktika, TalkPal) egyáltalán nem használ ismétléses tanulást, ez az egyik fő gyengeségük.

## Döntés
- **FSRS** algoritmus a **py-fsrs** könyvtárral a szerveren.
- Szókártyákra **és** nyelvtani pontokra is (a nyelvtani pont „értékelése” a feladatok és a beszélgetésbeli használat eredményéből számolódik).
- Hiteles forrás: szavaknál a `Card`, nyelvtani pontoknál az `ItemState` (ADR-0015).
- A mobil offline állapotban is értékelhet; az esedékességet a mobil is ki tudja számolni (Dart FSRS-implementáció), de **a szerver a hiteles forrás** (ADR-0010).

## Elvetett alternatívák
- **SM-2 (Anki klasszikus):** egyszerűbb, de az FSRS kevesebb ismétléssel ugyanazt a megtartást éri el.
- **Saját ad hoc ütemezés:** nincs ok újrafeltalálni.

## Következmények
- + Bevált, dokumentált algoritmus, kész implementációval.
- − A szerver és a mobil FSRS-implementációjának egyeznie kell (verzió rögzítése, a mobil számítása csak előzetes).
