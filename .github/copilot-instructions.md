# AI Agent Instructions for Panel_Docker

This repository is a minimal interactive dashboard built with [Panel](https://panel.holoviz.org/) and `hvplot`.
The entire application lives in a single script (`app.py`) and is designed to be containerized with Docker.

## Big picture

- **Single‑file web app**: `app.py` defines widgets, an interactive pipeline, and a `FastListTemplate` to layout the dashboard.
- **Data source**: uses Bokeh's built‑in `autompg_clean` sample dataset. No external APIs or databases.
- **Dependencies**: `panel`, `hvplot` (and transitively `bokeh`, `pandas` etc.). Listed in the top‑level `requirements` file.
- **Container**: `Dockerfile` builds from `python:3.9`, installs requirements, and runs `panel serve /code/app.py`.
  - Note: the `Dockerfile` copies `./requirements.txt` but the repo file is named `requirements` – keep them in sync or rename appropriately.
- **Deployment target**: originally written for Hugging Face Spaces; the CMD includes `--allow-websocket-origin` with a host string used by the Space.

## Developer workflows

1. **Local development**
   - Create & activate a virtual environment in the workspace root (e.g. `python -m venv .venv`).
   - Install dependencies:
     ```sh
     pip install -r requirements   # or rename to requirements.txt
     ```
   - Run interactively:
     ```sh
     panel serve app.py --show     # or just `python app.py` but `serve` is recommended
     ```
   - If you add a new package, update `requirements` and rebuild the container.

2. **Docker**
   - Build the image: `docker build -t panel-docker .`
   - Run: `docker run -p 7860:7860 panel-docker`
   - The CMD sets `--address 0.0.0.0 --port 7860`; adjust for other ports as needed.
   - Ensure the `--allow-websocket-origin` value matches the eventual deployment host.

3. **VS Code configuration**
   - The Python interpreter should point at the workspace `.venv` (use the status bar selector).
   - Pylance warnings about unresolved imports disappear once the env has `panel` installed.

## Code patterns & conventions

- Widgets are declared at module scope and then reused inside the reactive pipeline (`idf[…]`).
- The reactive DataFrame is built with `.interactive()` and filters using widget values directly.
- Plot creation uses `hvplot` chained off the pipeline result; pass widget objects (`yaxis`) to `hvplot`.
- UI layout uses `pn.template.FastListTemplate`; sidebar items mix strings and widget instances.
- `template.servable()` at the bottom makes the script usable by `panel serve`.

Agents modifying the code should keep everything in `app.py` unless refactoring into modules for larger features.

## Project-specific quirks

- **Requirements filename mismatch**: the Dockerfile expects `requirements.txt` but the repo contains `requirements`. Fixing this is a common change.
- **Hugging Face origin string**: if you change the deployment name, remember to update `--allow-websocket-origin`.
- The project has no tests or CI; new contributors should not assume a test harness exists.

## Files to reference

- `app.py`: primary application logic.
- `Dockerfile`: containerization steps and command-line arguments.
- `requirements` (or `requirements.txt`): dependency list.
- `README.md`: minimal metadata for HF Spaces; not a developer guide.


> **Next steps for AI agents:**
> - When editing the dashboard, focus on `app.py` and keep widget names consistent.
> - Update the dependency file and align Dockerfile when adding new packages.
> - Keep interactions simple: there are no async calls or background tasks.

Please review these notes and let me know if any sections feel unclear or if more details would be helpful.