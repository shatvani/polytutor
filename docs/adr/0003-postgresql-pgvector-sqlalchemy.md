# ADR-0003: PostgreSQL + pgvector, SQLAlchemy 2 + Alembic

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
Relációs adatok (kurzus, tudásállapot, naplók) és vektoros keresés (RAG a tankönyv szövegén) kell. A fejlesztő PostgreSQL-t futtat Dockerben.

## Döntés
- **PostgreSQL** a `pgvector` kiegészítővel (a `pgvector/pgvector` Docker-image).
- ORM: **SQLAlchemy 2.x** (deklaratív, típusos `Mapped[...]`), **async** driver.
  - ⚠️ **A driver a T1.6-os spike után véglegesítendő:** asyncpg vagy psycopg3 (`postgresql+psycopg`). A Procrastinate psycopg3-at használ; a feladat és az üzleti adat közös tranzakcióban való sorba állítása (ADR-0004) valószínűleg csak közös driverrel és kapcsolattal oldható meg.
- Migráció: **Alembic** (autogenerate + kézi ellenőrzés).
- Táblanév-előtag a modul neve (pl. `courses_unit`).
- Rugalmas szerkezetű adatok (pl. párbeszédsorok, FSRS-állapot) `JSONB`-ben.

## Elvetett alternatívák
- **Külön vektoradatbázis (Qdrant, Chroma):** plusz üzemeltetés, a méret (néhány könyv) nem indokolja.
- **SQLModel:** kényelmes, de kevésbé kiforrott; az SQLAlchemy tudás jobban átvihető.
- **Django ORM:** a Django választását nem tesszük meg (ADR-0001).

## Következmények
- + Egy adatbázis mindenre, egyszerű mentés.
- + Az SQLAlchemy a Data Engineer munkában is hasznos.
- − Az async SQLAlchemy tanulási görbéje meredekebb, mint az EF Core-é (session-kezelés, lazy loading tiltása async-ban).
