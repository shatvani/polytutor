# PolyTutor

Személyes, AI-alapú nyelvtanuló rendszer saját tananyagra (PDF + MP3), később e-könyv olvasóval.

## Dokumentáció
- [Specifikáció](docs/SPEC.md)
- [Architektúra-döntések (ADR)](docs/adr/README.md)
- [Feladatlista](docs/TASKS.md)
- [Tanulási jegyzetek](docs/notes/README.md)

## Szerkezet
| Mappa | Tartalom |
|---|---|
| `backend/` | Python/FastAPI backend (uv-projekt) |
| `docs/` | Specifikáció, ADR-ek, feladatok, jegyzetek |

## Fejlesztői környezet új gépen
1. Telepítendő: **git**, **[uv](https://docs.astral.sh/uv/)**, **VS Code**.
2. Git-azonosító (ha még nincs beállítva):
   ```
   git config --global user.name "<név>"
   git config --global user.email "<e-mail>"
   ```
3. Klónozás és függőségek:
   ```
   git clone <repo-url> polytutor
   cd polytutor/backend
   uv sync
   ```
   Az `uv sync` a hiányzó Python-verziót (`.python-version`) is letölti, és létrehozza a `.venv`-et.
4. VS Code: nyisd meg a repó gyökerét; a javasolt bővítményeket (`.vscode/extensions.json`) felajánlja telepítésre. Interpreter: `backend/.venv` (Windowson `Scripts\python.exe`, Linuxon/macOS-en `bin/python`).
5. Ellenőrzés a `backend` mappából:
   ```
   uv run python -c "import app.core, app.modules; print('ok')"
   ```
6. `.env`: a `.env.example` másolata, kitöltve (a T0.3-tól kell).
