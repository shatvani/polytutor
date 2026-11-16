# Jegyzet: repó-szerkezet, Python-csomagok, git-alapok

> Ellenőrizve: Python 3.13, uv 0.13.6, Windows – 2026-11-16
> Kapcsolódó feladat: TASKS T0.2

## Fogalmak – .NET-es párhuzammal

| Python | .NET | Megjegyzés |
|---|---|---|
| **modul** = egy `.py` fájl | osztályfájl + namespace | Az import egysége a fájl, nem az osztály. |
| **csomag** = mappa `__init__.py`-jal | namespace (+ mappa) | Nincs assembly; a mappaszerkezet maga a namespace. |
| `import` | `using` + assembly-hivatkozás | Az import **lefuttatja** a modult (első betöltéskor, egyszer). |
| `sys.path` | assembly probing path | Innen keresi a Python a csomagokat. |
| `__init__.py` lefutása | `[ModuleInitializer]` / statikus konstruktor | Ezért üres marad – nincs mellékhatás importkor. |
| `.gitkeep` | `<Folder Include="..." />` a `.csproj`-ban | Üres mappa megtartása (gitnél: csak konvenció). |

## `__init__.py`
- Csomaggá teszi a mappát → importálható (`from app.core import config`).
- Nélküle is importálható lenne (*namespace package*, Python 3.3+), de az több disztribúcióra bontott csomagokhoz való; a mypy/pytest/ruff a hagyományos csomagot kezeli kiszámíthatóan → **mindig legyen**.
- Az első importnál lefut, folyamatonként egyszer. Kipróbálva: egy `print` az `app/__init__.py`-ban az `import app; import app` hatására csak egyszer ír ki.
- Lehetne vele „kirakatot” (re-exportot) csinálni – **nálunk erre a `public.py` való** (ADR-0002), az `__init__.py` üres.

## Importok
```python
from app.modules.courses.public import get_unit   # abszolút – ezt használjuk
from .handler import handle                        # relatív – kerüljük
```
- Abszolút import mindenhol: egyértelmű, kereshető, a modulhatár-teszt (T0.6) könnyen ellenőrzi.
- ⚠️ **Körkörös import** futáskor `ImportError`-t adhat (C#-ban egy assemblyn belül ez nem gond). A `public.py`-szabály ez ellen is véd.
- `python -m app.main` (modulként, a csomaggyökérből) ≠ `python app/main.py` (fájlként – a csomagon belüli importok eltörhetnek).

## A kialakított szerkezet
```
polytutor/
├── .gitattributes, .gitignore, .env.example, README.md
├── .vscode/extensions.json     ← javasolt bővítmények
├── docs/
└── backend/
    ├── app/{__init__.py, main.py, core/__init__.py, modules/__init__.py}
    ├── tests/.gitkeep
    └── prompts/.gitkeep
```
- Modulmappák (`app/modules/<modul>/`) csak az első feature-rel jönnek létre – üres csomagot nem gyártunk előre.
- `mobile/`, `docker/` akkor jön létre, amikor tartalom kerül bele (a git üres mappát nem követ).
- Ellenőrzés: `uv run python -c "import app.core, app.modules; print('ok')"` (a `backend`-ből).

## `.env.example`
- `KULCS=érték` formában, **szóköz nélkül** az `=` körül – így a pydantic-settings, a Docker Compose és a shell is egyformán olvassa.
- Legyen sortörés a fájl végén (VS Code: `files.insertFinalNewline`).
- Változónként megjegyzés: ez a fájl dokumentáció is. Valódi érték csak a `.env`-be kerül.

## Sorvégek (Windows)
- `core.autocrlf=true` mellett a munkakönyvtárban CRLF van → Linux-konténerben (Docker, T0.4) a shell-szkript elszáll, a `.env`-értékek végén rejtett `\r` lehet.
- Megoldás: `.gitattributes` a repó gyökerében: `* text=auto eol=lf` – a repóval utazik, gépfüggetlen.
- Meglévő fájlok igazítása utána: `git add --renormalize .`

## Git-tanulság: az útvonalak relatívak
- `git add backend/ docs/` az **aktuális mappához** képest értelmez. A `backend` mappából futtatva nincs `backend/` alatta → `fatal: pathspec ... did not match` → a `git add` **semmit nem ad hozzá**, a commit elmarad.
- Megoldás: a repó gyökeréből dolgozz, vagy használd a `:/` előtagot (= repógyökér): `git add :/docs/notes`.
- Commit után mindig: `git log --oneline` – tényleg létrejött-e.
