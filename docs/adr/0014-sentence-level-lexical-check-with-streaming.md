# ADR-0014: Mondatonkénti lexikai ellenőrzés streaming mellett

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
Az ADR-0008 szerint a tanár válaszát generálás után spaCy-vel ellenőrizzük, és ha túl sok benne az új elem, egyszer újrageneráljuk (F-CNV-08). A SPEC 9 ugyanakkor streamelt választ kér, és az első tokennek 3 másodpercen belül meg kell érkeznie. Amit már kistreameltünk a kliensnek, azt nem lehet visszavonni. A két követelmény ebben a formában ütközik.

## Döntés
- Az LLM tokenfolyamát a szerver **mondatonként puffereli** (mondathatár-felismerés: spaCy `sentencizer` vagy egyszerű írásjel-szabály, a kész mondat után).
- Minden lezárt mondaton lefut a lexikai ellenőrzés (`app/core/nlp.py` szótövezés + összevetés az ismert anyaggal és a terv bővítési keretével). Az ellenőrzés a `conversation` modulban van.
- A kliens csak az **ellenőrzött mondatot** kapja meg (SSE-eseményként).
- Ha egy mondat túllépi a keretet: a generálás leáll, és a tanár **a már elküldött mondatokat folytatva**, egyszerűbb nyelvezetre utasítva újragenerálja a folytatást (beszélgetésenként fordulónként max. 1 újrapróbálás). Ha ez is túllépi a keretet, a mondat kimegy, és a túllépést naplózzuk (`Message.lexical_check`).
- A tanár nem ad vissza önbevallott lemmalistát; a használt elemek megállapítása kizárólag a spaCy-ellenőrzésé (ADR-0008 módosul).
- Hangos módban a TTS is mondatonként kapja a szöveget, így a hangkimenet is hamarabb indul.

## Elvetett alternatívák
- **Teljes puffer, majd ellenőrzés:** egyszerű, de nincs valódi streaming. Rövid válasznál is 2–4 másodperc késést jelent.
- **Azonnali streaming és utólagos ellenőrzés:** gyors, de az F-CNV-08 kontrollja elvész; csak a következő fordulóban lehetne javítani.

## Következmények
- + Az első mondat kb. 1–2 másodperc alatt megjelenik; a kontroll megmarad.
- + Mondatonkénti TTS-sel a hangos kör is gyorsul.
- − Mondathatár-felismerés kell streamelt szövegen (rövidítések, számok – pl. „z. B.”, „3.”).
- − A „folytatásos” újragenerálás promptja összetettebb, mint egy teljes újragenerálásé.
- Az első token célértéke (SPEC 9) **az első ellenőrzött mondatra** értendő.
