# ADR-0005: OpenAI portok mögött, API-kulcs csak a szerveren

**Állapot:** Elfogadva – 2026-11-15

## Kontextus
Kell LLM (beszélgetés, kinyerés, elemzés), STT, TTS és embedding. Az OpenAI egy szolgáltatónál mindet adja, jól beszél magyarul, németül, angolul, és támogatja a strukturált (Pydantic-sémás) kimenetet. A szolgáltató vagy a modell később cserélődhet.

## Döntés
- Az `ai` modulban **portok** (Python `Protocol`-ok): `LlmPort`, `SttPort`, `TtsPort`, `EmbeddingPort`. Első implementáció: **OpenAI Python SDK**.
- **Feladatonként konfigurálható modell** (pl. `AI_MODEL_CONVERSATION`, `AI_MODEL_FEEDBACK`, `AI_MODEL_EXTRACTION`, `AI_MODEL_STT`, `AI_TTS_VOICE_DE`, …) – modellnév sehol nincs a kódba égetve.
- Az API-kulcs **pydantic-settings**-szel töltődik: helyben `.env`, Dockerben környezeti változó / Docker secret. **A kulcs soha nem kerül a mobilra.**
- `ApiKeyProvider` absztrakció: most a konfigurációból olvas; később felhasználónkénti, titkosítva tárolt kulcsra bővíthető, globális tartalékkal.
- Minden hívás `AiUsage`-naplót ír (feature, modell, tokenek/hangmásodpercek, költségbecslés, prompt-verzió).
- Fontos: a ChatGPT-előfizetés **nem** fedezi az API-t; az API külön, használatalapú díjazású.

## Elvetett alternatívák
- **Közvetlen SDK-hívások a slice-okban:** szolgáltatócsere és tesztelés nehéz.
- **LangChain:** nagy absztrakciós réteg, ami ennél a méretnél inkább elrejti, mint egyszerűsíti a hívásokat; tanulási szempontból is jobb a közvetlen SDK a portok mögött.
- **Claude + külön beszédszolgáltató:** két szolgáltató a hanghoz.

## Következmények
- + Szolgáltató cserélhető, tesztekben a portok mockolhatók.
- + Költség követhető és korlátozható.
- − Saját absztrakciót kell karbantartani (kicsi).
