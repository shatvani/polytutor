# ADR-0016: Felhasználó-azonosító egyfelhasználós módban

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
A rendszer egyetlen felhasználónak készül, de a SPEC szerint a tervezés ne zárja ki a későbbi többfelhasználós működést. Ha a felhasználóhoz kötött táblák most nem kapnak `user_id`-t, később minden ilyen táblán adatmigrációra és a lekérdezések átírására lesz szükség.

## Döntés
- Az `identity` modulban `User` tábla (`identity_user`), **egyetlen sorral**, amelyet migráció vagy seed hoz létre.
- `user_id` kerül a felhasználóhoz kötött gyökértáblákba: `Enrollment`, `LanguageSettings`, `Book`, `WordKnowledge`, `DeviceToken`, `AiUsage`.
- A származtatott táblák (`ItemState`, `Session`, `Mistake`, `Card` stb.) az `Enrollment`-en vagy a `Book`-on keresztül kötődnek a felhasználóhoz; nem kapnak külön `user_id`-t.
- A kérés kontextusában egy `current_user` függőség (FastAPI `Depends`) adja a felhasználót. Most az admin-sessionből vagy az eszköz-tokenből mindig ugyanazt az egy felhasználót oldja fel.
- A modulok a `user_id`-t `identity/public.py`-on keresztül kapják; más modul nem olvassa a `User` táblát.

## Elvetett alternatívák
- **Nincs `user_id` (YAGNI):** most egyszerűbb, de a későbbi átállás sok táblát és lekérdezést érint.
- **Teljes többfelhasználós működés most:** regisztráció, jogosultságok, kulcskezelés – nem része a hatókörnek.

## Következmények
- + A későbbi többfelhasználós működés főként jogosultsági és UI-munka lesz, nem adatmigráció.
- − Minimális többletmunka: egy idegen kulcs és egy szűrőfeltétel a gyökértáblákban.
- A `Course` most nem kap tulajdonost; ha a kurzusok felhasználónkéntiek lesznek, külön döntés kell (és jogi felülvizsgálat, SPEC 9).
