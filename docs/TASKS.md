# PolyTutor – Feladatlista

> Kapcsolódó: [SPEC.md](SPEC.md), [ADR-ek](adr/)
>
> **Munkamenet minden feladatnál:**
> 1. Átbeszéljük a feladatot (Claude Code-dal, tervezési módban).
> 2. Megnézed a **🐍 Python-tanulnivalót** (Claude Code elmagyarázza, .NET-es párhuzammal).
> 3. **Te írod meg** a kódot.
> 4. Claude Code átnézi (helyesség, „pythonos”-e, tesztek).
> 5. Tanulási jegyzet a `docs/notes/`-ba (Claude Code vázolja, te átnézed).
> 6. Kipipálod, commit.
>
> Jelölések: `[ ]` nyitott · `[~]` folyamatban · `[x]` kész · **F-xxx** = SPEC-követelmény

---

## 0. fázis – Környezet és projektváz

- [ ] **T0.1 Python-fejlesztői környezet**
  - Python 3.12+, **uv**, VS Code / PyCharm + Python-kiegészítők, ruff, mypy.
  - 🐍 virtuális környezet (≈ projektszintű SDK), `pyproject.toml` (≈ `.csproj`), `uv add`, `uv run`.
  - ✅ Kész, ha: `uv run python -c "print('ok')"` fut.

- [ ] **T0.2 Repo-szerkezet**
  - `git init`, első commit a meglévő dokumentációval.
  - `backend/`, `mobile/` (később), `docs/`, `docker/`; `.gitignore`, `.env.example`, `README.md`.
  - Backend: `app/core/`, `app/modules/`, `tests/`, `prompts/` (ADR-0002).
  - 🐍 csomagok és modulok, `__init__.py`, abszolút importok.
  - ✅ Kész, ha: a mappaszerkezet az ADR-0002 szerint áll.

- [ ] **T0.3 FastAPI „hello” + konfiguráció**
  - `app/main.py`, `/health` végpont; `app/core/config.py` **pydantic-settings**-szel (`OPENAI_API_KEY`, `DATABASE_URL`, modellnevek).
  - 🐍 típusannotációk, Pydantic-modellek, `async def`, dependency injection FastAPI-ban (`Depends` ≈ konstruktor-injektálás).
  - ✅ Kész, ha: `/health` 200-at ad, a `/docs` (Swagger) megnyílik, a konfiguráció `.env`-ből töltődik.

- [ ] **T0.4 Docker Compose**
  - `pgvector/pgvector` PostgreSQL, backend (fejlesztői módban hot reloaddal), később worker.
  - ✅ Kész, ha: `docker compose up` után a backend eléri az adatbázist.

- [ ] **T0.5 Adatbázis-réteg**
  - `app/core/db.py`: async engine, session-kezelés (`Depends`-szel kérésenként); Alembic beállítása; `pgvector` kiterjesztés migrációval.
  - 🐍 SQLAlchemy 2 `Mapped[...]` modellek, async session, context manager (`async with` ≈ `using`).
  - ✅ Kész, ha: egy próbatábla migrációja oda-vissza fut.

- [ ] **T0.6 Tesztkeret és minőség**
  - pytest + pytest-asyncio, testcontainers-python (PostgreSQL), ruff + mypy, pre-commit.
  - **Modulhatár-teszt**: modul csak más modul `public.py`-jából importálhat (ADR-0002).
  - 🐍 pytest fixture-ök (≈ xUnit fixture/`IClassFixture`), `assert`, paraméterezett tesztek.
  - ✅ Kész, ha: `uv run pytest` zöld, a modulhatár-teszt egy szándékos szabálysértésen elbukik.

- [ ] **T0.7 Naplózás és hibakezelés**
  - Naplózókönyvtár kiválasztása (pl. structlog vagy a standard `logging` JSON-formázóval) – rövid indoklással.
  - Strukturált (JSON) log, kérésazonosító, egységes hibaválasz.
  - ✅ Kész, ha: egy dobott kivétel strukturált logbejegyzést és egységes JSON-hibát ad.

- [ ] **T0.8 CI** (GitHub Actions): ruff, mypy, pytest.

---

## 1. fázis – Walking skeleton, majd PDF-beolvasás (admin felületen)

> Sorrend: előbb egy **kézzel felvitt leckén** végigmenő, teljes tanulási kör (1C), és csak utána a PDF-beolvasás (1D). Az 1B kinyerési spike korán felméri a beolvasás kockázatát a saját könyveden.

### 1A. Alapok

- [ ] **T1.1 `ai` modul: portok és OpenAI-implementáció** (ADR-0005)
  - `LlmPort` (szöveg + strukturált kimenet Pydantic-sémával, streaming), `EmbeddingPort`; `AiUsage` napló (`prompt_version`-nel); `ApiKeyProvider`.
  - **Egyszerű havi kemény költséglimit** (konfigurációból): elérésekor a portok hívása hibát ad. (A részletes áttekintés a T4.9-ben jön.)
  - 🐍 `typing.Protocol` (≈ interface), generikusok, async generátorok (streaming ≈ `IAsyncEnumerable`).
  - ✅ Kész, ha: egy teszt mockolt porttal fut, egy kézi próba valódi API-val strukturált választ ad, és `AiUsage`-sor keletkezik; a limit túllépését teszt igazolja.

- [ ] **T1.2 Promptkezelés**
  - `prompts/<feature>/<verzió>.md` Jinja2-sablonok betöltése, verzió naplózása.
  - ✅ Kész, ha: a sablon változókkal renderelhető, a verzió bekerül az `AiUsage`-be.

- [ ] **T1.3 `identity`: felhasználó és admin bejelentkezés** (ADR-0016)
  - `User` tábla egyetlen sorral (seed/migráció), `current_user` függőség.
  - Egyetlen admin felhasználó (jelszó-hash a konfigurációban), session cookie.
  - ✅ Kész, ha: `/admin` csak bejelentkezve érhető el, és a `current_user` a seedelt felhasználót adja.

- [ ] **T1.4 Admin felület váza** (Jinja2 + HTMX, ADR-0006)
  - Alap layout, navigáció, HTMX betöltése, egy minta HTMX-es partial-frissítés.
  - ✅ Kész, ha: a menüpontok működnek, egy gomb HTMX-szel frissít egy részt.

### 1B. Kinyerési spike (kockázatfelmérés)

- [ ] **T1.5 PDF-kinyerési spike a saját könyvön** — F-ING-02, F-ING-03 előkészítése
  - Önálló script (nincs DB, nincs UI): PyMuPDF-szövegkinyerés 1–2 leckére, majd LLM-hívás a SPEC 8.2-es Pydantic-sémával; az eredmény JSON-fájlba kerül.
  - Tanulság rögzítése: mennyire jók a leckehatárok, a táblázatos oldalak, a szólisták; kell-e oldalkép. Ha a sémán módosítani kell, SPEC-frissítés.
  - 🐍 script vs. csomag (`if __name__ == "__main__":`), `pathlib`, Pydantic `model_validate_json`.
  - ✅ Kész, ha: 1–2 lecke kinyert JSON-ja kézzel átnézve, a tanulságok röviden leírva (pl. `docs/spikes/pdf-extraction.md`).

### 1C. Tanulási mag – walking skeleton

- [ ] **T1.6 `courses` modul: modell, migráció és seed** (SPEC 7)
  - Course, Unit, VocabularyItem, GrammarPoint, Dialogue, Exercise, TextPassage, ContentChunk.
  - **Seed-script**: egy lecke kézzel (vagy a T1.5 JSON-jából) felvíve, `approved=true` elemekkel.
  - ✅ Kész, ha: a migráció lefut, a seed betölt egy leckét, a `public.py` olvasó függvényei tesztelve.

- [ ] **T1.7 `learner`: Enrollment, ItemState, LanguageSettings** — F-LRN-01 (elölről), F-LRN-02, F-LRN-03, F-LRN-04
  - Beiratkozás elölről; tudásállapot létrehozása a közzétett itemekre; nyelvenkénti beállítások külön táblában (`user_id`-val, ADR-0016).

- [ ] **T1.8 Közös NLP-szolgáltatás: `app/core/nlp.py`** (ADR-0008, ADR-0013, ADR-0014)
  - spaCy de/en modellek, szótövezés, **mondatra bontás**, kivétellista (tulajdonnevek, számok).
  - ✅ Kész, ha: német és angol mondat szótövei és mondathatárai (rövidítésekkel, pl. „z. B.”) helyesen jönnek vissza tesztben.

- [ ] **T1.9 `planning`: SessionPlanner** — F-PLN-01…06
  - Tiszta függvény, 80/20/0 · 70/20/10 · 60/20/20 módok, súlyozás, `SessionPlan` mentése. Szóesedékesség a `vocabulary` public API-ból (ADR-0015; amíg nincs kártya, üres bemenet).
  - 🐍 `dataclass`-ok, tiszta függvények, property-alapú tesztelés (opcionális: hypothesis).
  - ✅ Kész, ha: unit-tesztek fedik az arányokat, a súlyozást és a módokat — AI nélkül.

- [ ] **T1.10 `conversation`: írásos beszélgetés** — F-CNV-01, F-CNV-06, F-CNV-07, F-CNV-09
  - Session + Message, tanár-prompt a tervből, streaming válasz (SSE).

- [ ] **T1.11 Mondatonkénti lexikai ellenőrzés** — F-CNV-08, ADR-0014
  - A tokenfolyam mondatonkénti pufferelése, a mondat szótöveinek összevetése az ismert anyaggal, csak az ellenőrzött mondat megy ki; keret-túllépésnél a folytatás egyszeri újragenerálása.
  - 🐍 async generátorok láncolása (≈ `IAsyncEnumerable` + LINQ-szerű pipeline).
  - ✅ Kész, ha: teszt bizonyítja, hogy túl sok új szó esetén újragenerálás történik, és a túllépő mondat nem kerül a kimenetre.

- [ ] **T1.12 Háttérfeladatok: Procrastinate + driver-spike** (ADR-0004, ADR-0003)
  - **Spike:** megoldható-e a közös tranzakció asyncpg-vel, vagy kell `postgresql+psycopg`? Az eredmény alapján az ADR-0003 véglegesítése.
  - Worker-folyamat, egy próbafeladat; Compose-ba worker-szolgáltatás.
  - ✅ Kész, ha: a feladat a kéréssel azonos tranzakcióban sorba áll, rollbacknél nem (teszt), és a worker lefuttatja.

- [ ] **T1.13 `feedback`: hibaelemzés** — F-FBK-01…05, F-FBK-07, F-FBK-08
  - Elemzés a tanár válaszával **párhuzamosan** (`asyncio.gather` ≈ `Task.WhenAll`); strukturált kimenet tervitem-kulcsokkal (SPEC 8.2); Mistake-mentés és ItemState-frissítés háttérfeladatként; javítási stílusok.

- [ ] **T1.14 Admin tesztbeszélgetés-oldal**
  - HTMX-es chat: kurzus/lecke választás, a `SessionPlan` megjelenítése, a beszélgetés, a javítások.
  - ✅ Kész, ha: **egy teljes gyakorlás végigvihető a böngészőben** a kézzel felvitt leckén. ← *1. fázis első mérföldköve (walking skeleton)*

### 1D. PDF-beolvasás

- [ ] **T1.15 Feltöltés és `SourceFile`** — F-ING-01
  - Fájltárolás (helyi kötet), SHA-256, kurzus létrehozása (nyelv, cím).
  - 🐍 fájlkezelés `pathlib`-bel, `UploadFile`.

- [ ] **T1.16 PDF-szövegkinyerés** — F-ING-02
  - PyMuPDF oldalanként; táblázatos oldalak felismerése (heurisztika) és oldalkép renderelése (a T1.5 tanulságai alapján).
  - ✅ Kész, ha: a mintakönyv oldalainak szövege és a jelölt oldalak képei előállnak.

- [ ] **T1.17 Leckehatárok és strukturált kinyerés** — F-ING-03, F-ING-04
  - Pydantic-kinyerési séma (SPEC 8.2), leckénkénti LLM-hívás háttérfeladatként, `IngestionJob` állapotkövetés.
  - ✅ Kész, ha: a mintakönyv első 3 leckéje `approved=false` elemekként bekerül.

- [ ] **T1.18 Átnéző felület** — F-ING-05
  - Leckénkénti nézet, elemek szerkesztése/törlése/felvétele, tömeges jóváhagyás (HTMX).
  - ✅ Kész, ha: egy teljes lecke kényelmesen átnézhető és közzétehető.

- [ ] **T1.19 Embedding (RAG-alap)** — F-ING-10
  - Szövegrészekre bontás, embedding (`embedding_model` rögzítésével), pgvector-index, hasonlósági keresés függvénye.
  - ✅ Kész, ha: egy nyelvtani kérdésre a releváns könyvbeli részlet jön vissza első találatként.
  - ✅ *1. fázis mérföldköve:* egy gyakorlás végigvihető a böngészőben a **saját könyvedből beolvasott** leckén.

---

## 2. fázis – Mobil (Flutter, Android), hanganyag, hang

### 2A. Backend előkészítés

- [ ] **T2.1 `/api/v1` JSON API a meglévő slice-okhoz** + eszköz-tokenes azonosítás (`DeviceToken`, párosítás az adminból, pl. QR-kóddal).
- [ ] **T2.2 Hálózati elérés**: Tailscale a fejlesztői gépen/szerveren; HTTPS.

### 2B. Flutter alapok

- [ ] **T2.3 Flutter-környezet** (Android Studio, emulátor, saját telefon fejlesztői módban).
  - 🎯 Dart-alapok: null safety, async/await, widgetek, `pubspec.yaml`.
- [ ] **T2.4 Projektváz**: Riverpod, dio, flutter_secure_storage; párosítás képernyő.
- [ ] **T2.5 Kurzus- és leckelista** képernyő.
- [ ] **T2.6 Írásos beszélgetés** képernyő (streaming, javítások megjelenítése, „Próbáld újra”) — F-FBK-03, F-FBK-04.

### 2C. Hanganyag (MP3)

- [ ] **T2.7 MP3-feltöltés és átírás** — F-ING-06, F-ING-07 (`SttPort`, időbélyegek).
- [ ] **T2.8 Automatikus párosítás** — F-ING-08 (rapidfuzz + embedding).
- [ ] **T2.9 Párosítás jóváhagyása / kézi hozzárendelés** az adminban — F-ING-09.

### 2D. Hangalapú beszélgetés

- [ ] **T2.10 `TtsPort` + TTS-gyorsítótár** (nyelvenkénti hang, sebesség).
- [ ] **T2.11 Hangpipeline a backendben** — F-CNV-02, F-CNV-03 (szó szerinti STT-utasítás; mondatonkénti TTS az ellenőrzött mondatokra, ADR-0014).
- [ ] **T2.12 Mobil: felvétel, STT-megerősítés, lejátszás** (record, just_audio).
- [ ] **T2.13 Gyakorlómódok** — F-CNV-04, F-CNV-05 (szerepjáték a könyv párbeszédeiből, szókincs-előkészítés).
  - ✅ *2. fázis mérföldköve:* hanggal végigvihető szerepjáték a telefonon, a könyv egy párbeszéde alapján.

---

## 3. fázis – Nyelvtan, feladatok, írás, hallás

- [ ] **T3.1 `grammar`: témalista és RAG-alapú magyarázat** — F-GRM-01…03
- [ ] **T3.2 `exercises`: könyvbeli feladatok megoldása** — F-EXR-01
- [ ] **T3.3 Generált feladattípusok + kiértékelés magyarázattal** — F-EXR-02, F-EXR-03
- [ ] **T3.4 Íráskorrekció** — F-EXR-04
- [ ] **T3.5 Hallásértés, diktálás, utánmondás** az eredeti MP3-akból — F-EXR-05
- [ ] **T3.6 „Ismételd, amíg hibátlan”** — F-EXR-07
- [ ] **T3.7 Mobil képernyők** a fentiekhez.

---

## 4. fázis – Szókártyák, szintfelmérés, visszacsatolás

- [ ] **T4.1 `vocabulary`: kártyák + FSRS** (py-fsrs) — F-VOC-01…03, ADR-0009, ADR-0015 (a kártya a hiteles forrás; `public.py`: esedékes kártyák a plannernek)
- [ ] **T4.2 Nyelvtani pontok FSRS-ütemezése** — F-GRM-04
- [ ] **T4.3 Offline szinkron** (drift, ReviewLog, idempotens sync végpont) — F-VOC-04, ADR-0010
- [ ] **T4.4 Leckék letöltése offline hallgatásra**
- [ ] **T4.5 Szintfelmérő a könyvön belül** — F-LRN-01 (placement mód)
- [ ] **T4.6 „Hibáim ismétlése” mód** — F-EXR-06
- [ ] **T4.7 Beszélgetés utáni jelentés** — F-FBK-06
- [ ] **T4.8 Esedékes szavak visszavezetése a beszélgetésbe** — F-VOC-05
- [ ] **T4.9 AI-költség áttekintés és havi keret** az adminban (SPEC 6, 9)

---

## 5. fázis – E-könyv olvasó (ADR-0013)

- [ ] **T5.1 EPUB-feltöltés és feldolgozás** (ebooklib, fejezetek, mondatok, szótövek) — F-RDR-01, F-RDR-02
- [ ] **T5.2 Olvasó nézet a mobilon** + szóra koppintva kontextusfüggő jelentés — F-RDR-03
- [ ] **T5.3 Mondatelemzés magyarul** — F-RDR-04
- [ ] **T5.4 Szó mentése kártyának** — F-RDR-05, F-RDR-10
- [ ] **T5.5 Felolvasás mondatkiemeléssel** — F-RDR-06
- [ ] **T5.6 Szóismeretség-követés és fejezetnehézség** — F-RDR-07
- [ ] **T5.7 Beszélgetés a fejezetről (spoiler-védelemmel)** — F-RDR-08
- [ ] **T5.8 Egyszerűsített bekezdés** — F-RDR-09

---

## Később (backlog)

- [ ] AI-generált, CEFR-alapú kurzus (könyv nélküli kezdés)
- [ ] Kiejtésértékelés – Azure (ADR-0011)
- [ ] Realtime beszéd-beszéd mód
- [ ] iOS-kiadás
- [ ] Szkennelt PDF (OCR)
- [ ] Többfelhasználós működés, felhasználónkénti API-kulcs
