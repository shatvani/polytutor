# ADR-0013: E-könyv olvasó külön tartalomtípusként

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
A felhasználó e-könyvet szeretne feltölteni, és az AI-val együtt olvasni: ismeretlen szavak, nyelvtan, szótárazás, beszélgetés a fejezetekről. Egy könyv nem lecke-szerkezetű tananyag: nincs benne szólista, nyelvtani pont vagy feladat.

## Döntés
- Külön `reader` modul saját modellel: `Book` → `Chapter` → `Sentence` (+ szótövek), `ReadingProgress`, `WordKnowledge`.
- **Nem** erőltetjük a `Course`/`Unit` modellbe.
- Közös szolgáltatások: a `vocabulary` (szókártya mentése) és a `learner` (tudásállapot, ha a könyv nyelve egy aktív kurzus nyelve) `public.py`-on keresztül.
- Formátum: elsőként DRM-mentes **EPUB** (ebooklib + HTML-tisztítás); később szöveges PDF/TXT.
- Fejezetről szóló beszélgetés RAG-gal, **csak a már elolvasott részre** korlátozva (spoiler-védelem).
- Ismert/ismeretlen szavak jelölése és a fejezet „nehézsége” a `WordKnowledge` alapján.

## Elvetett alternatívák
- **Könyv mint Course:** a kurzus-specifikus funkciók (feladatok, leckesorrend, 70/20/10) nem értelmezhetők rajta.

## Következmények
- + Tiszta modulhatár; az olvasó önállóan is fejleszthető (5. fázis).
- − A szótövezés és a szóismeretség-követés logikája részben átfed a `conversation` modul lexikai ellenőrzésével (ADR-0014) → a spaCy-alapú szótövezés közös, állapotmentes szolgáltatás: **`app/core/nlp.py`** (saját táblája nincs, ezért nem modul).
- Jogi megjegyzés: csak saját, jogszerűen birtokolt könyv, személyes célra.
