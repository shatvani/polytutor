# ADR-0008: Ellenőrzött bővítés – determinisztikus SessionPlanner

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
Az AI bővítheti a tananyagot, de ésszerű határok között: vissza kell nyúlnia korábbi leckékhez, és csak kevés új elemet hozhat be. Ha ezt egy prompt „hangulatára” bízzuk, kiszámíthatatlan és tesztelhetetlen lesz.

## Döntés
Négylépéses kör minden gyakorlásnál:
1. **SessionPlanner** – tiszta, determinisztikus Python-függvény (nincs benne AI). Bemenet: Enrollment, ItemState-ek, beállítások, idő. Kimenet: `SessionPlan` (fókusz-itemek, visszanyúlás-itemek, bővítési keret, helyzet).
   - Arány (mérsékelt mód): **70% aktuális lecke / 20% visszanyúlás / 10% új**. Szigorú: 80/20/0. Szabad: 60/20/20. Konfigurálható.
   - Visszanyúlás súlyozása: hibaarány, eltelt idő, kerülés, FSRS-esedékesség.
   - Új anyag: max. 3–5 új szó; új nyelvtan csak kész kifejezésként (chunk).
2. **LLM végrehajtás** – a terv a promptba kerül; a tanár célzottan előhozza a célszerkezeteket. (Önbevallott lemmalistát nem ad vissza; a használt elemeket a 3. lépés állapítja meg.)
3. **Lexikai ellenőrzés** – a válasz **spaCy**-szótövezése, összevetés az ismert lemmákkal; keret túllépése esetén egy újragenerálás egyszerűbb válaszért. Streaming mellett mondatonként fut (ADR-0014).
4. **Visszacsatolás** – a feedback modul frissíti az ItemState-eket (helyes használat, hiba, kerülés).

## Elvetett alternatívák
- **Csak prompt-alapú szabályozás:** nem tesztelhető, nem következetes.
- **Csak a modell önbevallása a használt szavakról:** megbízhatatlan; a spaCy-ellenőrzés független. Ezért az önbevallást egyáltalán nem kérjük.

## Következmények
- + A tanulási logika unit-tesztelhető AI nélkül.
- + A `SessionPlan` mentése miatt utólag látszik, mit és miért gyakoroltatott a rendszer.
- − spaCy nyelvi modellek kellenek (de/en), plusz memória a konténerben.
- − A tulajdonnevek, számok, nemzetközi szavak kezelésére kivétellista kell.
