# ADR-0001: Python + FastAPI backend

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
A fejlesztő tapasztalt .NET-fejlesztő, aki Data Engineer, majd AI Engineer irányba képzi át magát. A projekt legnehezebb részei (PDF-feldolgozás, szótövezés, hangfeldolgozás, embedding, FSRS) a Python-ökoszisztémában a legérettebbek. A fejlesztő maga írja a kódot, és közben tanulja a Pythont; a lassabb haladást vállalja.

## Döntés
A backend **Python 3.12+**, **FastAPI**, **Pydantic v2**, csomagkezelés **uv**, linter/formatter **ruff**, típusellenőrzés **mypy** (strict módban), tesztek **pytest**.

## Elvetett alternatívák
- **ASP.NET Core (.NET):** gyorsabb haladás, ismert eszköztár (Wolverine, EF Core), de gyengébb NLP/PDF-eszközök, és nem szolgálja az átképzést.
- **Vegyes (.NET + Python sidecar):** két nyelv, két futtatókörnyezet egy embernek; közel a mikroszolgáltatásokhoz.
- **Django:** túl sok beépített konvenció; a FastAPI közelebb áll a Minimal API szemlélethez.

## Következmények
- + Közvetlen hozzáférés: spaCy, PyMuPDF, py-fsrs, OpenAI Python SDK, Pydantic-alapú strukturált kimenet.
- + A projekt portfólióértékű az átképzéshez.
- − Az első hetek lassabbak; a .NET-es minták Python-megfelelőit meg kell tanulni.
- − A Wolverine-féle üzenetkezelésnek nincs egy az egyben megfelelője (lásd ADR-0004).
