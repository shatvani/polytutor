# ADR-0002: Moduláris monolit, Vertical Slice szervezés

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
Egyfelhasználós, egy fejlesztős projekt; a mikroszolgáltatások indokolatlanok. A fejlesztő a Vertical Slice Architecture-t és a moduláris monolitot ismeri és preferálja.

## Döntés
- Egy telepíthető FastAPI-alkalmazás, egy adatbázis.
- Modulok az `app/modules/<modul>/` alatt (lista: SPEC 4. fejezet).
- Modulon belül **feature-mappák** (slice-ok): `app/modules/<modul>/features/<feature_név>/` – benne: `endpoint.py` (router), `handler.py` (logika), `schemas.py` (Pydantic be/ki), opcionálisan `views.py` (HTML-kimenet az adminnak).
- Közös modulrészek: `models.py` (SQLAlchemy), `public.py` (más moduloknak kiajánlott szolgáltatásfelület), `events.py`.
- **Szabály:** modul nem importál más modul `models.py`-jából és `features`-éből; csak `public.py`-ból. Ezt egy egyszerű teszt (import-linter vagy saját pytest-ellenőrzés) őrzi.
- Közös infrastruktúra: `app/core/` (config, db, logging, security).

## Elvetett alternatívák
- **Rétegzett (controllers/services/repositories) felépítés:** szétszórja egy funkció kódját.
- **Mikroszolgáltatások:** üzemeltetési teher, semmi előny egy felhasználónál.

## Következmények
- + Egy funkció egy helyen; jól illeszkedik a Claude Code-os, feladatonkénti munkához.
- + A modulhatárok később akár külön csomaggá is kiemelhetők.
- − Fegyelem kell a `public.py`-szabály betartásához (ezért automatikus ellenőrzés).
