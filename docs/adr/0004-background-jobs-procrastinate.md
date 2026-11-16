# ADR-0004: Háttérfeladatok – Procrastinate

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
Hosszú vagy aszinkron feladatok: PDF-kinyerés, MP3-átírás és -párosítás, embedding-készítés, mondatonkénti hibaelemzés, e-könyv-feldolgozás. .NET-ben ezt a Wolverine (üzenetek, outbox) adná.

## Döntés
**Procrastinate**: PostgreSQL-alapú feladatsor Pythonhoz.
- A feladat ugyanabban a tranzakcióban sorba állítható, mint az üzleti adat (outbox-szerű garancia).
- Külön worker-folyamat (ugyanabból a kódbázisból, külön konténerben vagy processzben).
- Újrapróbálás és állapotkövetés a feladatokra.

## Elvetett alternatívák
- **Celery / RQ / arq:** Redis vagy RabbitMQ kellene mellé; több üzemeltetés.
- **FastAPI BackgroundTasks:** nincs perzisztencia, újraindításkor elveszik.
- **Saját `jobs` tábla + polling:** a Procrastinate pont ezt adja készen.

## Következmények
- + Nincs új infrastruktúra-elem, a PostgreSQL elég.
- + Tranzakcionális sorba állítás.
- − Kisebb közösség, mint a Celeryé; ha a projekt nagyot nő, felülvizsgálandó.
