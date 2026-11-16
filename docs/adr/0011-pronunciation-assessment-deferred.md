# ADR-0011: Kiejtésértékelés későbbre halasztva (Azure AI Speech)

**Állapot:** Javasolt – 2026-11-15

## Kontextus
Az OpenAI STT szöveget ad, nem kiejtési pontszámot, és hajlamos a rossz kiejtést is helyes szóként leírni. Mért, hangonkénti kiejtésértékelést (pontosság, folyékonyság, teljesség, hanglejtés) az **Azure AI Speech Pronunciation Assessment** ad, angolra és németre is. Az LLM-alapú „hallgasd meg és véleményezd” megoldás nem mért és nem következetes.

## Döntés
- Az első fázisokban **nincs** kiejtésértékelés.
- A `media` modulban előkészítünk egy `PronunciationAssessorPort`-ot; az első implementáció később az Azure lesz, az utánmondás-feladatokhoz (F-EXR-05) és a „Ismételd, amíg hibátlan” módhoz kötve.
- A visszajelzés mindig **magyar nyelvű útmutatással** jár (mi a hiba, hogyan javítsd), nem csak pontszámmal.

## Következmények
- + Az első fázisok egy szolgáltatóval (OpenAI) mennek.
- − Később második szolgáltató, második kulcs, külön költség.
