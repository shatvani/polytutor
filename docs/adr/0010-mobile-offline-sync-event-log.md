# ADR-0010: Mobil offline szinkron eseménynaplóval

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
Szókártyázás és a letöltött hanganyag hallgatása net nélkül is kell (pl. utazás közben). Egy felhasználó, de lehet több eszköz.

## Döntés
- A mobil helyi SQLite-ban (drift) tárolja: letöltött kurzus-pillanatkép (kártyák, hangfájl-hivatkozások), és az offline **értékelési események** (`ReviewLog`: card_id, rating, reviewed_at, device_id, `client_event_id`).
- Szinkron: a mobil feltölti az eseményeket (`POST /api/v1/vocabulary/reviews:sync`); a szerver **idempotensen** (client_event_id alapján) beírja és időrendben újraszámolja az FSRS-állapotot; válaszul a friss kártyaállapotokat adja.
- A kártyák tartalmát csak a szerver módosítja (az adminban); a mobil csak olvas és eseményt ír → nincs valódi ütközés.
- Hangfájlok: kifejezett „letöltés offline használatra” leckénként.

## Elvetett alternatívák
- **Állapot-szinkron (last-write-wins):** több eszköznél elveszhetnek értékelések.
- **Teljes offline-first adatbázis-replikáció (pl. CouchDB-szerű):** túlméretezett.

## Következmények
- + Egyszerű, ütközésmentes, auditálható.
- − Az FSRS-újraszámolás késve érkező eseményeknél a kártya állapotát utólag módosíthatja (elfogadható).
- Az offline mód először csak a szókártyákra és a letöltött hanganyagra vonatkozik; az AI-funkciók hálózatot igényelnek.
