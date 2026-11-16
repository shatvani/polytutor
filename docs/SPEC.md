# PolyTutor – Specifikáció

> Munkanév: **PolyTutor** (szabadon átnevezhető).
> Verzió: 0.2 – 2026-11-15 (dokumentum-átnézés: ADR-0014–0016, adatmodell-pontosítások)
> Kapcsolódó dokumentumok: [ADR-ek](adr/), [TASKS.md](TASKS.md)

---

## 1. Cél és hatókör

Személyes, AI-alapú nyelvtanuló rendszer, amely **a felhasználó saját tananyagára épül**. A felhasználó feltölti a tankönyvét (PDF + MP3), a rendszer ebből strukturált kurzust épít, és az AI ezen a kurzuson belül tanít: beszélget, nyelvtant magyaráz, szavakat kérdez ki, írást és párbeszédeket gyakoroltat. Később e-könyvek közös olvasását is támogatja.

**Elsődleges felhasználó:** egyetlen felhasználó (a fejlesztő maga). A tervezés ne zárja ki a későbbi többfelhasználós működést, de azt most nem valósítjuk meg.

**Nyelvek:** bármely nyelv, amelyet az AI-szolgáltató támogat; első célnyelvek: **német** és **angol**. A magyarázatok nyelve: **magyar**.

### 1.1 Nem célok (tudatosan kizárva)

- Gamifikáció: nincs streak, napi cél, pontverseny, bűntudatkeltő értesítés.
- Avatárok, animált tanár-karakterek.
- Merev tanulási útvonal: a felhasználó bármikor válthat leckét, nyelvet, módot.
- Saját, beépített tananyag az első körben (AI-generált kurzus csak később, lásd 12. fejezet).
- iOS-támogatás az első körben (Android először).

---

## 2. Fogalomtár

| Fogalom | Jelentés |
|---|---|
| **Course** | Egy tankönyvből (vagy később AI-generálásból) készült kurzus egy adott nyelvre. |
| **Unit** | Egy lecke a kurzuson belül, sorrenddel. |
| **Item** | A lecke tanulható eleme: szó (`VocabularyItem`), nyelvtani pont (`GrammarPoint`), párbeszéd (`Dialogue`), feladat (`Exercise`), szöveg (`TextPassage`). |
| **Enrollment** | A felhasználó beiratkozása egy kurzusra; tárolja, hol tart. |
| **ItemState** | A felhasználó tudásállapota egy itemről (elsajátítás, hibák, következő esedékesség). |
| **SessionPlan** | A determinisztikus tervező által egy gyakorláshoz összeállított „óravázlat”. |
| **Session** | Egy gyakorlás (beszélgetés, feladatsor, olvasás). |
| **Mistake** | Egy rögzített hiba kategóriával és magyarázattal. |
| **Ingestion** | Tananyag-beolvasás: PDF/MP3 → strukturált kurzus, emberi jóváhagyással. |
| **Book** | Feltöltött e-könyv olvasáshoz (a kurzustól független tartalomtípus). |
| **Expansion** | Az AI által a tananyagon túl behozott új elemek (ellenőrzött bővítés). |

---

## 3. Felületek

### 3.1 Admin webfelület (asztali böngésző)
FastAPI + Jinja2 + HTMX. Feladatai:
- Tananyag feltöltése (PDF, MP3), beolvasás indítása, állapotának követése.
- **Beolvasás átnézése és javítása** (leckék, szavak, nyelvtani pontok, párbeszédek, feladatok).
- MP3-sávok párosításának jóváhagyása / átrendezése.
- E-könyv feltöltése.
- Beállítások: nyelvenkénti javítási stílus, bővítési mód, AI-modellek.
- Eszköz-tokenek kezelése (mobil párosítás).
- AI-használat és költség áttekintése, havi keret.
- **Fejlesztői tesztfelület**: egyszerű írásos beszélgetés-oldal, hogy a tanulási logika a mobilapp elkészülte előtt is kipróbálható legyen.

### 3.2 Mobilalkalmazás (Flutter, Android)
A tanuló felülete:
- Kurzus- és leckeválasztás, haladás áttekintése.
- Beszélgetés írásban és hanggal.
- Feladatok, nyelvtani leckék, írásgyakorlat.
- Hallás utáni értés, diktálás, utánmondás (az eredeti MP3-akból).
- Szókártyák (SRS) – **offline is**.
- E-könyv olvasó (későbbi fázis).

### 3.3 Backend API
FastAPI JSON API a mobilnak (`/api/v1/...`), HTML-végpontok az adminnak (`/admin/...`). A slice-ok üzleti logikája közös; csak a kimeneti réteg különbözik.

---

## 4. Modulok (moduláris monolit)

| Modul | Felelősség |
|---|---|
| `courses` | Kurzusmodell (Course, Unit, Itemek), olvasó lekérdezések. |
| `ingestion` | PDF-feldolgozás, LLM-alapú kinyerés, átnézési munkafolyamat. |
| `media` | Hangfájlok, átírás, párosítás, TTS-gyorsítótár. |
| `learner` | Enrollment, ItemState, szintfelmérés a könyvön belül, beállítások. |
| `planning` | SessionPlanner: determinisztikus óravázlat-készítés (70/20/10). |
| `conversation` | Beszélgetés (írás/hang), szerepjáték, gyakorlómódok. |
| `feedback` | Mondatelemzés, hibarögzítés, beszélgetés utáni jelentés. |
| `grammar` | Nyelvtani pontok böngészése, magyarázat a könyv alapján. |
| `exercises` | Könyvbeli és generált feladatok, kiértékelés, „Hibáim ismétlése”. |
| `vocabulary` | Szókártyák, FSRS-ütemezés, offline szinkron. |
| `reader` | E-könyvek, olvasási munkamenet, szótárazás, beszélgetés a fejezetről. |
| `ai` | AI-szolgáltató absztrakció (LLM, STT, TTS, embedding), használatnaplózás. |
| `identity` | Felhasználó (egyfelhasználós mód, ADR-0016), admin bejelentkezés, eszköz-tokenek. |

**Modulhatár-szabály:** modul nem nyúl más modul tábláihoz közvetlenül. Más modult csak annak publikus szolgáltatásfelületén (`<modul>/public.py`) vagy eseményen keresztül ér el.

**Közös, állapotmentes szolgáltatás:** `app/core/nlp.py` – spaCy-alapú szótövezés és mondatra bontás, kivétellistával; használja a `conversation` (lexikai ellenőrzés) és a `reader` (ADR-0013).

---

## 5. Funkcionális követelmények

Az azonosítók a TASKS.md-ben hivatkozva vannak.

### 5.1 Tananyag-beolvasás (`ingestion`)

- **F-ING-01** PDF feltöltése egy új kurzushoz (nyelv, cím, forrásmegjelölés megadásával).
- **F-ING-02** Szöveges PDF szövegkinyerése oldalanként (PyMuPDF). Táblázatos/összetett oldalaknál az oldal képe is a modellhez kerül.
- **F-ING-03** Leckehatárok felismerése (tartalomjegyzék + LLM), majd leckénkénti strukturált kinyerés Pydantic-sémába (lásd 8.2).
- **F-ING-04** A kinyerés háttérfeladatként fut, állapota az adminban követhető (várakozik / fut / átnézésre vár / hibás).
- **F-ING-05** **Kötelező átnézés**: a kinyert elemek csak jóváhagyás után válnak tanulhatóvá. Az admin felületen minden elem szerkeszthető, törölhető, új elem felvehető.
- **F-ING-06** MP3-ak feltöltése a kurzushoz (több fájl egyszerre).
- **F-ING-07** MP3-ak átírása időbélyegekkel (STT).
- **F-ING-08** Automatikus párosítás tartalom alapján: az átiratot a párbeszédekhez/szövegekhez hasonlítjuk (szöveges egyezés + embedding-hasonlóság). Párosítatlan sáv a sávsorrend alapján kap javasolt leckét, önálló hallásértési anyagként.
- **F-ING-09** A párosítás jóváhagyása / módosítása az adminban.
- **F-ING-10** Beágyazások (embedding) készítése a lecke szövegrészeiről pgvectorba, a könyvre támaszkodó magyarázatokhoz.

### 5.2 Tanulói állapot és beiratkozás (`learner`)

- **F-LRN-01** Beiratkozás egy kurzusra két módon: **elölről** (1. lecke) vagy **szintfelmérő után** (a felmérés javasol kezdő leckét).
- **F-LRN-02** A szint nincs előre beállítva; a haladás a tényleges teljesítményből számolódik, kézzel bármikor felülírható, a haladás elvesztése nélkül.
- **F-LRN-03** Minden itemhez `ItemState`: elsajátítási pontszám, helyes használatok, hibák, kerülés-számláló, utolsó gyakorlás; nyelvtani pontoknál FSRS-esedékesség is (szavaknál az esedékesség a kártyán van, ADR-0015).
- **F-LRN-04** Nyelvenkénti beállítások (`LanguageSettings`, nem kurzusonként): javítási stílus, bővítési mód, új szavak maximuma, TTS-sebesség.

### 5.3 Gyakorlásterv (`planning`)

- **F-PLN-01** Minden gyakorlás előtt a SessionPlanner **determinisztikus Python-kóddal** (nem LLM-mel) összeállítja a `SessionPlan`-t.
- **F-PLN-02** Tartalomarány (mérsékelt mód, alapértelmezés): **70%** aktuális lecke, **20%** visszanyúlás korábbi leckékhez, **10%** új anyag a könyvön túl.
- **F-PLN-03** A visszanyúlás súlyozása: hibaarány (magas → előbb), utolsó gyakorlás óta eltelt idő, kerülés (sosem használt szerkezet), FSRS-esedékesség.
- **F-PLN-04** Új anyag korlátai: beszélgetésenként legfeljebb **3–5 új szó** (témához illő, gyakori); **új nyelvtani szerkezet nem** vezethető be a könyv előtt, kivéve **kész kifejezésként** (chunk), magyarázattal.
- **F-PLN-05** Bővítési módok: *Szigorú* (80/20/0), *Mérsékelt* (70/20/10), *Szabad* (60/20/20). Az arányok konfigurálhatók.
- **F-PLN-06** A `SessionPlan` mentésre kerül (utólag visszanézhető, mit gyakoroltatott a rendszer).

### 5.4 Beszélgetés (`conversation`)

- **F-CNV-01** Írásos beszélgetés az AI-tanárral a `SessionPlan` alapján.
- **F-CNV-02** Hangalapú beszélgetés pipeline-nal: felvétel → STT → (megerősítés) → LLM → TTS.
- **F-CNV-03** **STT-megerősítés**: a felismert szöveg küldés előtt megjelenik és javítható. Az STT-t utasítjuk a szó szerinti, hibákat megőrző átírásra.
- **F-CNV-04** Gyakorlómódok: szabad beszélgetés, szerepjáték (a könyv párbeszédeiből vagy helyzetekből), vita (B1-től ajánlott), mondatépítés, „telefonhívás” (csak hang).
- **F-CNV-05** Szerepjáték előtt **szókincs-előkészítés**: kulcsszavak és mondatpanelek megjelenítése.
- **F-CNV-06** „Elakadtam” lehetőség: a felhasználó magyarul is írhat/mondhat, az AI megérti és segít célnyelven megfogalmazni.
- **F-CNV-07** A tanár szándékosan olyan helyzeteket teremt, amelyek a terv célszerkezeteit igénylik.
- **F-CNV-08** **Lexikai ellenőrzés**: a tanár válaszát **mondatonként**, a kliensnek küldés előtt spaCy-szótövezéssel összevetjük az ismert anyaggal; ha az új elemek túllépik a keretet, a folytatást egyszerűbben újrageneráljuk (max. 1 újrapróbálás). Részletek: ADR-0014.
- **F-CNV-09** Teljes átirat, utólag visszanézhető; szóra koppintva fordítás.

### 5.5 Visszajelzés (`feedback`)

- **F-FBK-01** Minden felhasználói mondat elemzése **a tanár válaszának generálásával párhuzamosan** (nem lassítja a beszélgetést, de a javítás a válasz előtt megjeleníthető, F-FBK-03). A `Mistake` mentése és az `ItemState` frissítése háttérfeladat.
- **F-FBK-02** Javítási stílusok (nyelvenként állítható): *Finom* (csak a beszélgetés végén / jelentős hibánál), *Minden hiba azonnal*, *Csak a végén*. Kezdő szinten javasolt alapértelmezés: minden hiba azonnal.
- **F-FBK-03** A javítás csak hibás mondatnál jelenik meg (nincs „Feedback” gomb minden sornál). Opció: a javítás a tanár válasza **előtt** látható.
- **F-FBK-04** „Próbáld újra”: a javított mondat újramondása/újraírása.
- **F-FBK-05** Hibakategóriák (bővíthető lista): igeidő, ragozás, névelő, eset, szórend, elöljárószó, szóválasztás, helyesírás, kiejtés (később).
- **F-FBK-06** Beszélgetés utáni jelentés: kategorizált hibalista, erősségek, 1–3 javasolt gyakorlás.
- **F-FBK-07** A hibák frissítik az érintett itemek `ItemState`-jét (visszacsatolás a planninghez).
- **F-FBK-08** „Rossz magyarázat” jelölés bármely AI-magyarázatra; naplózva.

### 5.6 Nyelvtan (`grammar`)

- **F-GRM-01** Böngészhető nyelvtani témalista leckénként.
- **F-GRM-02** Magyar nyelvű magyarázat a **könyv saját magyarázatára támaszkodva** (RAG), példákkal.
- **F-GRM-03** Bármikor kérhető magyarázat tetszőleges, a kurzusban szereplő nyelvtani pontra.
- **F-GRM-04** Nyelvtani pontokra is vonatkozik az ismétlési ütemezés (FSRS).

### 5.7 Feladatok (`exercises`)

- **F-EXR-01** A könyv kinyert feladatai megoldhatók az appban.
- **F-EXR-02** Generált feladattípusok: kiegészítés, fordítás oda-vissza, szórend, feleletválasztás, ragozás.
- **F-EXR-03** Minden válasz kiértékelése **magyarázattal** (miért helyes/helytelen).
- **F-EXR-04** Íráskorrekció: a lecke témájához kapcsolódó írásfeladat, javítással és magyarázattal.
- **F-EXR-05** Hallásértés (kérdések a hanganyaghoz), diktálás (eltérés-kiemeléssel), utánmondás mondatonként.
- **F-EXR-06** **„Hibáim ismétlése”** mód: a rögzített hibákból generált célzott feladatsor.
- **F-EXR-07** „Ismételd, amíg hibátlan” mondatgyakorlás.

### 5.8 Szókincs (`vocabulary`)

- **F-VOC-01** Szókártyák a kurzus szavaiból és a felhasználó által mentett szavakból (beszélgetés, olvasás).
- **F-VOC-02** FSRS-ütemezés; a szavak ütemezésének hiteles forrása a kártya (ADR-0015).
- **F-VOC-03** Kártya: szó (névelővel, többes számmal, ahol releváns), jelentés, példamondat (lehetőleg a könyvből), hang (TTS vagy eredeti).
- **F-VOC-04** **Offline ismétlés** a mobilon; szinkronizálás eseménynaplóval (lásd ADR-0010).
- **F-VOC-05** Az esedékes/gyenge szavak a planning révén visszakerülnek a beszélgetésekbe.

### 5.9 E-könyv olvasó (`reader`) – későbbi fázis

- **F-RDR-01** DRM-mentes EPUB feltöltése (később: szöveges PDF, TXT). Csak jogszerűen birtokolt, saját célra használt könyv.
- **F-RDR-02** Feldolgozás: fejezetek, bekezdések, mondatok; a mondatok szótövezése (spaCy).
- **F-RDR-03** Olvasó nézet a mobilon; **szóra koppintva** kontextusfüggő jelentés (szótári alak, nem, ragozás, a mondatbeli jelentés).
- **F-RDR-04** **Mondatra koppintva** nyelvtani elemzés magyarul.
- **F-RDR-05** Szó mentése szókártyának egy mozdulattal (a könyvbeli mondattal mint példával).
- **F-RDR-06** Felolvasás (TTS) mondatonkénti kiemeléssel; sebesség állítható.
- **F-RDR-07** Szóismeretség-követés: ismert / tanulás alatt / ismeretlen; az oldalon színnel jelölve, és a fejezet „nehézsége” (ismeretlen szavak aránya) kijelezve.
- **F-RDR-08** Beszélgetés a fejezetről: összefoglalás, szövegértési kérdések, vélemény, szereplők – a fejezet szövegére támaszkodva (RAG), a már olvasott részre korlátozva (spoiler-védelem).
- **F-RDR-09** Opcionális egyszerűsített változat egy bekezdésről a felhasználó szintjén.
- **F-RDR-10** Ha a könyv nyelve egy aktív kurzus nyelve, a mentett szavak és hibák ugyanabba a tudásállapotba kerülnek (`vocabulary`, `learner`).

---

## 6. Beállítások és alapértékek

A nyelvenkénti beállítások a `learner` modul `LanguageSettings` táblájában vannak (nem az `Enrollment`-ben), így ugyanazon nyelv több kurzusa közös beállítást használ.

| Beállítás | Hatókör | Alapérték |
|---|---|---|
| Javítási stílus | nyelvenként | kezdő kurzusnál: *minden hiba azonnal*; egyébként: *finom* |
| Bővítési mód | nyelvenként | *Mérsékelt* (70/20/10) |
| Új szavak max. / beszélgetés | nyelvenként | 5 |
| TTS-sebesség | nyelvenként | 1,0 (kezdő kurzusnál 0,85) |
| Javítás a válasz előtt | globális | igen |
| Havi AI-költségkeret | globális | konfigurálható; elérésekor figyelmeztetés, 120%-nál tiltás (egyszerű kemény limit már az 1. fázistól) |

---

## 7. Adatmodell (vázlat)

> Minden tábla: `id` (UUID), `created_at`, `updated_at`. A modul-előtag a táblanévben (pl. `courses_unit`).

**courses**
- `Course`: language (ISO 639-1), title, source_type (`uploaded` | `generated`), source_note, status (`draft` | `in_review` | `published`)
- `Unit`: course_id, order, title, topic, summary
- `VocabularyItem`: unit_id, lemma, display_form, article/gender, plural, part_of_speech, meaning_hu, example_sentence, example_source (`book` | `ai`), approved
- `GrammarPoint`: unit_id, title, explanation_book (kinyert), explanation_hu, examples, approved
- `Dialogue`: unit_id, title, lines (JSON: speaker, text), audio_track_id (nullable), approved
- `Exercise`: unit_id, type, instruction, content (JSON), solution (JSON), source (`book` | `generated`), approved
  - A `content`/`solution` szerkezete `type`-onként eltér: a kódban Pydantic **diszkriminált unió** írja le (`type` a diszkriminátor), a JSONB csak a tárolás.
- `TextPassage`: unit_id, title, text, audio_track_id (nullable), approved
- `ContentChunk`: course_id, unit_id, text, embedding (pgvector), embedding_model
  - Az `embedding_model` rögzíti, melyik modell készítette. Modellcsere (más dimenzió) esetén újraszámolás és migráció kell.

**ingestion**
- `IngestionJob`: course_id, kind (`pdf` | `audio`), status, error, started_at, finished_at
- `SourceFile`: course_id, kind, original_name, storage_path, sha256, page_count / duration

**media**
- `AudioTrack`: source_file_id, order, transcript, segments (JSON: start, end, text), matched_entity_type, matched_entity_id, match_score, match_status (`suggested` | `confirmed` | `unmatched`)
- `TtsCache`: text_hash, voice, speed, storage_path

**learner**
- `Enrollment`: user_id, course_id, start_mode (`scratch` | `placement`), current_unit_id
- `LanguageSettings`: user_id, language, correction_style, expansion_mode, expansion_ratios (JSON), max_new_words, tts_speed
- `ItemState`: enrollment_id, item_type, item_id, mastery (0–1), correct_uses, errors, avoided_count, last_practiced_at, fsrs_state (JSON, csak nyelvtani pontnál), due_at (csak nyelvtani pontnál)
  - Az `item_type` + `item_id` polimorf hivatkozás a `courses` itemjeire: **szándékosan nincs FK** (modulhatár, ADR-0002). A létezést a `courses/public.py` ellenőrzi; törölt itemnél az `ItemState` árva marad, és a planner kihagyja.
  - Szavaknál az ütemezés a `Card`-on van (ADR-0015).

**planning**
- `SessionPlan`: enrollment_id, mode, focus_items (JSON), review_items (JSON), expansion_budget (JSON), scenario (JSON)

**conversation**
- `Session`: enrollment_id, plan_id, mode, started_at, ended_at, report (JSON)
- `Message`: session_id, role, text, input_mode (`text` | `voice`), audio_path, stt_raw, stt_confirmed, lexical_check (JSON)

**feedback**
- `Mistake`: message_id, enrollment_id, original, corrected, explanation_hu, category, item_refs (JSON), flagged_wrong (bool)
  - Az `item_refs` itemazonosítókat tartalmaz (`item_type` + `item_id`), nem szöveges címeket. A hibaelemző prompt a `SessionPlan` itemjeit rövid kulccsal kapja meg, és ezekre hivatkozik (lásd 8.2).

**vocabulary**
- `Card`: enrollment_id (nullable), book_id (nullable), vocabulary_item_id (nullable), lemma, language, front, back, example, audio_path, fsrs_state (JSON), due_at
  - Az `fsrs_state`/`due_at` a szó ütemezésének hiteles forrása, a `ReviewLog`-ból számolva (ADR-0015, ADR-0010).
  - A `vocabulary_item_id` a könyvbeli szónál van kitöltve; az olvasóból vagy beszélgetésből mentett szónál csak a `lemma`.
- `ReviewLog`: card_id, rating, reviewed_at, device_id, client_event_id (idempotencia)

**reader**
- `Book`: user_id, language, title, author, source_file_id, status
- `Chapter`: book_id, order, title
- `Sentence`: chapter_id, order, text, lemmas (JSON)
- `ReadingProgress`: book_id, chapter_id, sentence_order
- `WordKnowledge`: user_id, language, lemma, status (`known` | `learning` | `unknown`)

**ai**
- `AiUsage`: user_id, feature, provider, model, prompt_version, input_tokens, output_tokens, audio_seconds, cost_estimate

**identity**
- `User`: display_name (egyfelhasználós módban egyetlen sor, ADR-0016)
- `DeviceToken`: user_id, name, token_hash, last_seen_at, revoked

---

## 8. AI-integráció

### 8.1 Hívások és felelősségek

| Funkció | Típus | Megjegyzés |
|---|---|---|
| Leckekinyerés | LLM, strukturált kimenet | Pydantic-séma; oldalkép opcionálisan |
| Beszélgetés (tanár) | LLM, streaming | `SessionPlan` a promptban; mondatonkénti lexikai ellenőrzés (ADR-0014) |
| Hibaelemzés | LLM, strukturált kimenet | külön hívás, alacsony temperature; olcsóbb modell is lehet |
| Feladatgenerálás és -kiértékelés | LLM, strukturált kimenet | |
| Nyelvtani magyarázat | LLM + RAG | a könyv szövegrészeire támaszkodva |
| Beszédfelismerés | STT | szó szerinti átírás, időbélyegek |
| Felolvasás | TTS | gyorsítótárazva |
| Embedding | Embedding | pgvector |

Minden hívás a `ai` modul portjain (`LlmPort`, `SttPort`, `TtsPort`, `EmbeddingPort`) keresztül megy; az első implementáció OpenAI. **A modellnevek konfigurációból** jönnek feladatonként (nem beégetve).

### 8.2 Strukturált kimenetek (mintasémák)

**Leckekinyerés (rövidítve):**
```json
{
  "unit": { "order": 6, "title": "Freizeit", "topic": "szabadidő" },
  "vocabulary": [
    { "lemma": "können", "display_form": "können", "part_of_speech": "verb",
      "meaning_hu": "tud, képes", "example_sentence": "Ich kann gut schwimmen." }
  ],
  "grammar_points": [
    { "title": "Modalverben: können, müssen, wollen", "explanation_book": "...", "examples": ["..."] }
  ],
  "dialogues": [ { "title": "Am Wochenende", "lines": [ { "speaker": "Anna", "text": "..." } ] } ],
  "exercises": [ { "type": "gap_fill", "instruction": "...", "content": {}, "solution": {} } ]
}
```

**Hibaelemzés:**
```json
{
  "has_errors": true,
  "corrected": "Ich habe gestern Fußball gespielt.",
  "natural_alternative": "Gestern habe ich Fußball gespielt.",
  "errors": [
    { "fragment": "Ich habe gestern gespielt Fußball", "category": "word_order",
      "explanation_hu": "Perfektben a befejezett melléknévi igenév (gespielt) a mondat végére kerül.",
      "item_ref": "G2" }
  ],
  "used_items": ["G2", "V7", "V12"],
  "avoided_targets": []
}
```
> Az `item_ref`, a `used_items` és az `avoided_targets` a promptban átadott tervitemek rövid kulcsai (pl. `G2` = 2. nyelvtani pont, `V7` = 7. szó a tervben). A kód a kulcsokat `item_type` + `item_id` párra oldja fel; ismeretlen kulcsot eldob és naplóz.

### 8.3 Promptkezelés
- A promptok verziózott fájlokban (`prompts/<feature>/<version>.md`), Jinja2-sablonként; a kódban nincs hosszú szöveges prompt.
- Minden AI-hívás naplózza: feature, prompt-verzió, modell, tokenek, költségbecslés.

---

## 9. Nem-funkcionális követelmények

- **Biztonság:** az OpenAI API-kulcs **csak a szerveren** van (env/secret); a mobil a saját backendet hívja eszköz-tokennel. Admin felület jelszavas bejelentkezéssel. Elérés otthoni hálózaton kívülről Tailscale-en keresztül (nincs nyilvános port).
- **Konfiguráció:** pydantic-settings; `.env` nincs verziókezelve; `.env.example` igen.
- **Költség:** `AiUsage` napló, havi keret, TTS-gyorsítótár.
- **Teljesítmény:** az írásos válasz első **ellenőrzött mondata** < 3 s (streaming, ADR-0014); hangos kör (STT+LLM+TTS, mondatonkénti TTS) célérték < 5 s az első hangig.
- **Tesztelhetőség:** a SessionPlanner és a lexikai ellenőrzés tiszta függvények, AI nélkül unit-tesztelhetők. AI-hívások tesztekben mockolva; integrációs tesztek Testcontainers PostgreSQL-lel.
- **Adatvédelem / jog:** csak saját, jogszerűen birtokolt tananyag és könyv, személyes célra. Többfelhasználós működés előtt jogi felülvizsgálat szükséges.
- **Mentés:** PostgreSQL napi dump + a feltöltött fájlok könyvtára.
- **Naplózás:** strukturált (JSON) log; hibák AI-hívásonként visszakereshetők.

---

## 10. Ütemezés (fázisok)

| Fázis | Tartalom |
|---|---|
| **0** | Fejlesztői környezet, projektváz, Docker Compose, CI-alapok |
| **1** | Kurzusmodell, SessionPlanner, írásos beszélgetés + visszajelzés kézzel felvitt leckén (admin tesztfelületen) – *walking skeleton*; utána PDF-beolvasás + átnézés |
| **2** | Flutter alapok (Android): párosítás, kurzuslista, írásos beszélgetés; MP3-beolvasás és -párosítás; hangalapú beszélgetés |
| **3** | Nyelvtan, feladatok, íráskorrekció, hallásértés, diktálás, utánmondás |
| **4** | Szókártyák (FSRS) offline szinkronnal, szintfelmérő a könyvön belül, „Hibáim ismétlése”, beszélgetés utáni jelentés |
| **5** | E-könyv olvasó |
| **Később** | AI-generált kurzus, Azure kiejtésértékelés, Realtime API, iOS, szkennelt PDF (OCR) |

Részletek: [TASKS.md](TASKS.md).

---

## 11. Nyitott kérdések

1. A párosítatlan MP3-sávok kezelése elég-e sorrend alapján, vagy kell kézi „sáv → lecke” hozzárendelő felület már a 2. fázisban? (Javaslat: igen, egyszerű legördülő.)
2. Szintfelmérő a könyvön belül: hány kérdés / lecke? (Javaslat: leckénként 3 kérdés, megállás az első 2 hibás leckénél.)
3. E-könyvnél a PDF-formátum támogatása szükséges-e az EPUB mellett?
4. Projekt végleges neve.

---

## 12. Későbbi bővítések (nem része a jelenlegi hatókörnek)

- AI-generált, CEFR-alapú kurzus (könyv nélküli kezdéshez).
- Kiejtésértékelés (Azure AI Speech Pronunciation Assessment) – ADR-0011.
- Realtime beszéd-beszéd mód.
- iOS-kiadás.
- Szkennelt PDF OCR-rel.
- Többfelhasználós működés felhasználónkénti API-kulccsal.
