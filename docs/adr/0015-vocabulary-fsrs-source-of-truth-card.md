# ADR-0015: A szókincs FSRS-állapotának hiteles forrása a `Card`

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
A SPEC 7 korábbi változatában a szavak ütemezése két helyen jelent meg: a `learner` modul `ItemState.fsrs_state`/`due_at` mezőiben és a `vocabulary` modul `Card` + `ReviewLog` párosában. Az ADR-0010 szerint az offline értékelések a `ReviewLog`-ba kerülnek, és ebből számolódik újra a kártya állapota. Két forrásnál nem egyértelmű, melyik az igaz, és a szinkron eltérést okozhatna.

## Döntés
- A szavak FSRS-állapota (`fsrs_state`, `due_at`) **csak a `Card`-on** van, a `ReviewLog`-ból számolva (ADR-0009, ADR-0010).
- Az `ItemState` a **használatot** követi: mastery, helyes használatok, hibák, kerülés, utolsó gyakorlás. Szavakra nem tárol FSRS-állapotot.
- Nyelvtani pontokra (F-GRM-04) nincs kártya, ezért az ő FSRS-állapotuk az `ItemState`-ben marad (`fsrs_state`, `due_at` csak `item_type = grammar_point` esetén).
- A planner a szavak esedékességét a `vocabulary/public.py`-on keresztül kérdezi le (pl. `get_due_cards(enrollment_id, now)`).
- A könyvön kívüli (olvasóból, beszélgetésből mentett) szavak kártyája `ItemState` nélkül is létezhet.

## Elvetett alternatívák
- **Minden FSRS-állapot az `ItemState`-ben:** egy helyen lenne, de a `vocabulary` modul offline szinkronja (ADR-0010) a `learner` tábláit írná, és a könyvön kívüli szavakhoz is `ItemState` kellene.

## Következmények
- + Egy szónak egy ütemezése van; az offline szinkron egyetlen modulon belül marad.
- + Tiszta modulhatár: a `learner` nem függ a kártyáktól.
- − A planner két forrásból dolgozik (ItemState + `vocabulary` public API); a bemenetét a hívó állítja össze, így a planner tiszta függvény marad.
