# ADR-0007: Tananyag-beolvasás kötelező emberi átnézéssel

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
A rendszer a felhasználó tankönyvére épül (szöveges PDF + MP3, a sávok nincsenek jelölve a könyvben). Az LLM-alapú kinyerés nem lesz hibátlan; egy hibás szókártya vagy nyelvtani magyarázat rosszat tanít.

## Döntés
**PDF:**
1. Szövegkinyerés oldalanként **PyMuPDF**-fel; az összetett (táblázatos) oldalak képe is a modellhez kerül.
2. Leckehatárok: tartalomjegyzék + LLM.
3. Leckénként strukturált kinyerés Pydantic-sémába (SPEC 8.2).
4. Minden kinyert elem `approved = false` állapotban jön létre; **csak jóváhagyás után tanulható**.
5. Szövegrészek embeddingje pgvectorba (RAG).

**MP3:**
1. Átírás időbélyegekkel (STT).
2. Párosítás tartalom alapján: normalizált szöveges hasonlóság (pl. rapidfuzz) + embedding-hasonlóság a párbeszédekkel/szövegekkel.
3. Párosítatlan sáv → javasolt lecke a sávsorrend alapján, önálló hallásértési anyagként.
4. Jóváhagyás/átrendezés az adminban.

## Elvetett alternatívák
- **Teljesen automatikus beolvasás:** gyors, de a hibák csendben bekerülnek a tanulásba.
- **Kézi felvitel:** pontos, de egy teljes könyvnél aránytalanul sok munka.

## Következmények
- + Megbízható tananyag; a felhasználó közben át is nézi a leckét (önmagában is tanulás).
- − Beolvasásonként kézi munka; az átnéző felületnek kényelmesnek kell lennie (tömeges jóváhagyás, gyors szerkesztés).
- Jogi megjegyzés: csak saját, jogszerűen birtokolt anyag, személyes célra.
