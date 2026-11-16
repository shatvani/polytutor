# Jegyzet: Python-projekt létrehozása uv-vel

> Ellenőrizve: uv 0.13.6, Windows, PowerShell – 2026-11-16
> Kapcsolódó feladat: TASKS T0.1

## Mi mire jó – .NET-es párhuzammal

| Python / uv | .NET | Megjegyzés |
|---|---|---|
| CPython interpreter | .NET runtime | Nincs külön fordítás; a `.pyc` csak gyorsítótár. |
| `.python-version` | `global.json` | A projekt Python-verzióját rögzíti. |
| `.venv` (virtuális környezet) | – | A csomagok az interpreter mellé települnek, nem globális gyorsítótárból hivatkozódnak; ezért kell projektenként külön környezet. |
| `pyproject.toml` | `.csproj` | Függőségek + eszközbeállítások (ruff, mypy, pytest). |
| `uv.lock` | `packages.lock.json` | Commitolandó. |
| `uv add` / `uv add --dev` | `dotnet add package` | `--dev`: csak fejlesztéshez kell. |
| `uv sync` | `dotnet restore` | |
| `uv run <parancs>` | `dotnet run` | A `.venv`-et nem kell aktiválni. |
| ruff | Roslyn-analizátorok + `dotnet format` | Linter és formázó egyben. |
| mypy | a C#-fordító típusellenőrzése | ⚠️ Futás közben senki nem ellenőrzi a típusokat – csak a mypy. |

## Lépések

### 1. Python-verzió telepítése
```powershell
uv python install 3.13
```
Az uv saját Pythont tölt le; a gépre telepített más verziókhoz (pl. 3.14) nem nyúl. Lista: `uv python list`.

### 2. Projekt létrehozása
```powershell
cd C:\Users\hatvanis\polytutor
mkdir backend
cd backend
uv init --app --no-package --python 3.13 --name polytutor
```
Létrejön: `pyproject.toml`, `.python-version` (`3.13`), `main.py`, `README.md`.

⚠️ **A `--no-package` kell.** Nélküle (és csak `--app`-pal is) az uv 0.13 csomag-sablont készít: `src/<név>/__init__.py`, `[build-system]` és `[project.scripts]` a `pyproject.toml`-ban. Ez könyvtárhoz való, egy futtatott alkalmazáshoz felesleges.

Ellenőrizd: ne legyen `src/` mappa, és ne jöjjön létre beágyazott `.git`.

### 3. Fejlesztői eszközök
```powershell
uv add --dev ruff mypy
```
Létrejön a `.venv` és az `uv.lock`; a `pyproject.toml`-ba bekerül a `[dependency-groups]` → `dev` lista.

### 4. VS Code
```powershell
code --install-extension ms-python.python
code --install-extension charliermarsh.ruff
code --install-extension ms-python.mypy-type-checker
```
(A Pylance és a Python Environments a `ms-python.python`-nal együtt települ.)

Interpreter kiválasztása:
1. Nyiss meg egy `.py` fájlt – a státuszsor csak ilyenkor mutatja az interpretert.
2. Kattints a jobb alsó sarokban a „Python” feliratra (vagy Ctrl+Shift+P → *Python: Select Interpreter*).
3. Válaszd: `backend\.venv\Scripts\python.exe`.

Windowson a venv programjai a `Scripts\`-ben vannak (Linuxon/macOS-en: `bin/`).

Siker esetén a státuszsorban ez látszik: `polytutor (3.13.x)` – a projekt neve és a verzió.

### 5. Ellenőrzés (a `backend` mappából)
```powershell
uv run python -c "print('ok')"
uv run python --version     # 3.13.x – nem a rendszer Pythonja
uv run ruff --version
uv run mypy --version
```
VS Code-ban: `x: int = "szöveg"` sor egy `.py` fájlban → piros aláhúzás = a típusellenőrzés működik.

## Mi kerül gitbe?
- ✅ `pyproject.toml`, `uv.lock`, `.python-version`, `README.md`
- ❌ `.venv/`, `.mypy_cache/`, `.ruff_cache/`, `__pycache__/` (`.gitignore`)

## Ha már csomag-sablonnal jött létre
1. Töröld az `src/` mappát.
2. A `pyproject.toml`-ból töröld a `[build-system]` és a `[project.scripts]` szakaszt.
3. `uv sync` – ez a `.venv`-ből is eltávolítja a telepített saját csomagot.
