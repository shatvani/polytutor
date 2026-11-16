# ADR-0006: Admin – Jinja2 + HTMX; tanulói kliens – Flutter (Android)

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
Két eltérő felhasználási helyzet: (1) adminisztráció asztali böngészőből (feltöltés, átnézés, táblázatok, űrlapok); (2) tanulás telefonon (hang, offline szókártyák). A fejlesztő mobilalkalmazást szeretne, Flutterrel, először Androidra.

## Döntés
- **Admin:** szerveroldali renderelés **Jinja2**-vel + **HTMX** (részleges frissítés, SSE a streameléshez). Ugyanabban a FastAPI-alkalmazásban, `/admin` alatt.
- **Tanulói kliens:** **Flutter** (Dart), **Android** először. Javasolt csomagok (a 2. fázisban véglegesítendő): Riverpod (állapot), dio (HTTP), drift (helyi SQLite, offline), record (hangfelvétel), just_audio (lejátszás), flutter_secure_storage (eszköz-token).
- **API-first:** a mobil a `/api/v1` JSON API-t használja; az OpenAPI-leírás a FastAPI-ból automatikusan generálódik, ebből Dart-kliens generálható.
- **Áthidalás:** amíg a mobilapp nincs kész (1. fázis), az admin felületen egy egyszerű **tesztbeszélgetés-oldal** szolgál a tanulási logika kipróbálására.

## Elvetett alternatívák
- **PWA először:** nem kell harmadik nyelv, de a fejlesztő a natív appot választotta; offline és hangkezelés Flutterben megbízhatóbb.
- **.NET MAUI:** ismerős C#, de vegyes stabilitási tapasztalatok.
- **React Native / Expo:** TypeScript-ökoszisztéma; a Flutter hang- és offline-támogatása ennél a projektnél előnyösebb, és a fejlesztő a Fluttert választotta.
- **Python mobil (Kivy, BeeWare, Flet):** nem elég érett hangkezeléshez és éles használathoz.

## Következmények
- + Az admin gyorsan elkészíthető, nem kell hozzá SPA.
- + Natív mobilélmény, offline lehetőség.
- − A Dart a harmadik nyelv (Python mellett); a Flutter-munka ezért csak a backend 1. fázisa után indul.
- − Az iOS később külön munka (Apple fejlesztői fiók, tesztelés).
