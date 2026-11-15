# HoneyMoon: Lunar Landing Site Navigator

**Live service:** https://nasa-space-apps-seoul-two.vercel.app

<img width="1160" height="648" alt="image" src="https://github.com/user-attachments/assets/ec1d6312-eb1d-429d-b8ac-6b246271639b" />

HoneyMoon ranks landing sites at the lunar south pole and shows the best arrival day for each one. The user describes a trip, and the backend checks every hour of the planned stay for 72,297 candidate sites. This repository holds the backend and the frontend. It was built by Team 샤브샤브 for the NASA Space Apps Challenge 2026 (Seoul), challenge "Lunar Landing Site Navigator".

## Team
 
| Team Leader | Team Member | Team Member |
| :---: | :---: | :---: |
| <img src="https://github.com/user-attachments/assets/d6c9689c-8822-47ee-9fde-d3fd7b6bb201" width="100" alt="Jimin Shin"> | <img src="https://github.com/user-attachments/assets/9ee29677-d817-464f-9d60-0aa6419e3087" width="100" alt="Sanghyun Kim"> | <img src="https://github.com/user-attachments/assets/4714c90b-c2d1-4266-83e6-2ac5c915b5be" width="100" alt="Youngeun Seol"> |
| Jimin Shin | Sanghyun Kim | Youngeun Seol |
| Frontend | Aerospace Engineer | Backend |
| [zziminally](https://github.com/zziminally) | [sanghyunKimsam](https://github.com/sanghyunKimsam) | [0euun](https://github.com/0euun) |

## Overview and implemented scope

### Tech stack

**Backend**  
![Python 3.12](https://img.shields.io/badge/Python_3.12-3776AB?style=flat-square&logo=python&logoColor=white) ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white) ![NumPy](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=numpy&logoColor=white) ![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=flat-square&logo=scipy&logoColor=white) ![Numba](https://img.shields.io/badge/Numba-00A3E0?style=flat-square&logo=numba&logoColor=white) ![Rasterio](https://img.shields.io/badge/Rasterio-5B8C51?style=flat-square) ![GeoPandas](https://img.shields.io/badge/GeoPandas-139C5A?style=flat-square&logo=pandas&logoColor=white) ![Shapely](https://img.shields.io/badge/Shapely-3F7F3F?style=flat-square) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![Railway](https://img.shields.io/badge/Railway-0B0D0E?style=flat-square&logo=railway&logoColor=white)

**Frontend**  
![React 19](https://img.shields.io/badge/React_19-61DAFB?style=flat-square&logo=react&logoColor=black) ![Vite 8](https://img.shields.io/badge/Vite_8-646CFF?style=flat-square&logo=vite&logoColor=white) ![Three.js](https://img.shields.io/badge/Three.js-000000?style=flat-square&logo=threedotjs&logoColor=white) ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) ![Node.js 22.12+](https://img.shields.io/badge/Node.js_22.12%2B-5FA04E?style=flat-square&logo=nodedotjs&logoColor=white) ![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white)

**Data and imagery**  
![NASA PGDA](https://img.shields.io/badge/NASA_PGDA-0B3D91?style=flat-square&logo=nasa&logoColor=white) ![JPL Horizons](https://img.shields.io/badge/JPL_Horizons-0B3D91?style=flat-square&logo=nasa&logoColor=white) ![NASA Moon Trek](https://img.shields.io/badge/NASA_Moon_Trek-0B3D91?style=flat-square&logo=nasa&logoColor=white) ![USGS](https://img.shields.io/badge/USGS-00264D?style=flat-square) ![MIT Illumination Maps](https://img.shields.io/badge/MIT_Illumination_Maps-A31F34?style=flat-square)

### How it works

1. The backend traces each site's skyline from the 80 m LOLA elevation model: 720 directions, up to 100 km, for observer heights of 1, 5 and 10 m. These horizon masks are computed in advance.
2. A table of hourly Sun and Earth positions from JPL Horizons (2026-01-01 to 2028-12-31, 26,304 entries) is shared by all sites.
3. For each request, the backend checks only the hours of the stay and the landing-date search window. The Sun or Earth counts as visible when it is above the site's skyline in its direction.
4. Each site gets five scores (communication, flatness, roughness, sunlight, mission objective) weighted by the user's importance ratings. Communication and sunlight scores include the longest gap, so one three-day outage scores lower than three short ones.

### Implemented

Frontend (`frontend/`, React 19, Vite 8, Three.js):

- Mission configuration: landing date, stay duration, mission type (crewed or uncrewed), science objective, power and communication needs, slope limit, Sun and Earth visibility criteria, observer height, and landing-date search range.
- Importance weighting: Communication, Flatness, Roughness, Sunlight and objective-specific criteria rated from 1 to 5, with the resulting weights shown.
- Candidate sites: ranked sites and a suitability grid on a south pole map. Zooming and panning refresh the recommendations for the visible region.
- Site details in three tabs: Overview (scores, exclusion reasons, best landing date), 3D Terrain (surrounding elevation surface), and Sun & Earth (hourly visibility timeline).
- Moon locator: a lunar globe that shows where the selected area lies.

Backend (`backend/`, FastAPI):

- A 1 km base grid south of 85°S with 72,297 sites, plus refined analysis of the visible region.
- Exclusion rules: slope limit (default 5°, range 0–15°), crewed sunlight constraint, and uncrewed battery endurance and maximum communication blackout.
- Sun criteria `PARTIAL`, `HALF_DISK`, `FULL` (default `HALF_DISK`). Earth criteria `UPPER_EDGE`, `CENTER_DELTA`, `FULL` (default `CENTER_DELTA`, margin 0.25°).
- Objective scores for `POLAR_VOLATILES` (distance to the nearest permanently shadowed region), `PLANETARY_PROCESSES` (number of geological units within 20 km) and `IMPACT_HISTORY` (geology plus distance to craters of 20 km or more).
- Landing-date search of 0–30 days around the chosen date (default 7).
- Endpoints: `GET /health`, `GET /config`, `GET /weights`, `POST /analyses`, and under `/analyses/{analysisId}`: `/recommendations`, `/grid`, `/sites/{siteId}`, `/sites/{siteId}/visibility`, `/sites/{siteId}/terrain`.
- Validation against NASA/MIT average Sun and Earth maps at 2,000 sites: correlation 0.984 (Sun) and 0.980 (Earth). Details are in `backend/docs/DECISIONS.md`.

## Screenshots

**Search**
<img width="2880" height="1980" alt="image" src="https://github.com/user-attachments/assets/8475bdfa-cb6b-4766-974c-0ee9df5d99d1" />

**Compare**
<img width="2880" height="1980" alt="image" src="https://github.com/user-attachments/assets/0db54b7e-c0a8-43db-8b73-55bebe0c9fe3" />

**See why**
<img width="2880" height="1980" alt="image" src="https://github.com/user-attachments/assets/b5a892c4-6434-4b8f-928f-c9290cf6bead" />

## Installation and run

Run the backend first, then the frontend. Paths assume the backend is in `backend/` and the frontend is in `frontend/`.

### Requirements

| Component | Requirement |
| --- | --- |
| Backend | Python 3.12 |
| Frontend | Node.js 22.12 or later and npm |
| Both | Internet access (source data, JPL Horizons, map imagery) |
| Optional | PostgreSQL 18 (persistent analysis storage), Docker (container builds) |

The backend needs source datasets that are not stored in Git (see [Source data](#source-data)).

### 1. Backend

Install (from `backend/`):

```powershell
# Windows PowerShell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements-dev.txt
```

```sh
# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements-dev.txt
```

For a runtime-only installation, use `python -m pip install -r requirements.txt`.

Create `.env` from the example, then replace its contents with the values in [Backend variables](#backend-variables):

```powershell
Copy-Item .env.example .env
```

```sh
cp .env.example .env
```

Download the DEM and the Sun/Earth baseline maps, then prepare the remaining source data as described in [Source data](#source-data):

```sh
python scripts/download_data.py
```

Generate the analysis artifacts, in this order (horizon mask generation is computationally intensive):

```sh
python -m scripts.fetch_sky_table
python -m scripts.precompute_grid
python -m scripts.precompute_reference
python -m scripts.precompute_masks
```

Start the server:

```sh
uvicorn app.main:app --reload
```

Verify:

- API documentation: http://127.0.0.1:8000/docs
- Health and loaded sky data range: http://127.0.0.1:8000/health

`/health` returns `DATA_NOT_READY` until all required artifacts are loaded. A missing objective artifact blocks analysis for that objective.

Backend checks:

```sh
python -m pytest
ruff check .
```

### 2. Frontend

From `frontend/`:

```sh
npm install
```

Create `.env.local` and set the backend address:

```powershell
Copy-Item .env.example .env.local
```

```sh
cp .env.example .env.local
```

```dotenv
VITE_API_BASE_URL=http://127.0.0.1:8000
```

Start the dev server and open the Local URL it prints (normally http://localhost:5173). Configure a mission and select **Analyze**.

```sh
npm run dev
```

Restart the dev server after changing `VITE_API_BASE_URL`. Frontend checks:

```sh
npm test
npm run lint
npm run build
```

## Environment variables, data and settings

### Backend variables

Minimal `.env` for local development without PostgreSQL:

```dotenv
DATA_DIR=./data
ARTIFACTS_DIR=./artifacts
CORS_ORIGINS=http://localhost:3000
WEB_CONCURRENCY=1
```

Remove the empty `DATABASE_URL` line from `.env` when you run without PostgreSQL. To keep analyses across restarts, create a PostgreSQL database and add:

```dotenv
DATABASE_URL=postgresql://postgres:YOUR_PASSWORD@localhost:5432/lunar
```

Replace the user name, password, host, port and database name with your own. The PostgreSQL user needs permission to create the analysis table. Do not commit `.env`.

| Variable | Default | Purpose |
| --- | --- | --- |
| `DATA_DIR` | `./data` | Location of the source datasets. |
| `ARTIFACTS_DIR` | `./artifacts` | Location of the precomputed grid, horizon masks and other analysis files. |
| `DATABASE_URL` | unset | PostgreSQL connection URL. When unset, analyses are kept in an in-memory cache and are lost on restart. |
| `WEB_CONCURRENCY` | `1` | Uvicorn worker count in the container. Keep it at `1`; local caches and refined-site updates are not synchronized across workers. |
| `CORS_ORIGINS` | `http://localhost:3000` | Declared, but the app currently uses the constant in `app/settings.py` (`"*"`), so this variable does not restrict origins. |
| `PORT` | `8000` | Port for the container start command. Railway sets it. |

`scripts/download_data.py` reads `DATA_DIR` from the process environment, not from `.env`. With a different directory, set it in the shell first (for example `$env:DATA_DIR="D:\lunar-data"` in PowerShell). `scripts/fetch_sky_table.py` and the reference script use the default paths; if you change `DATA_DIR` or `ARTIFACTS_DIR`, move the generated files to the configured locations.

### Frontend variables

| Variable | Purpose |
| --- | --- |
| `VITE_API_BASE_URL` | Base URL of the backend API, for example `http://127.0.0.1:8000` for a local backend. |

### Source data

`scripts/download_data.py` downloads only the DEM and the baseline files. Prepare the rest by hand so that `DATA_DIR` has this layout:

```text
data/
  dem/
    LDEM_80S_80MPP_ADJ.TIF
  baseline/
    AVGVISIB_85S_060M_201608.IMG
    AVGVISIB_85S_060M_201608.LBL
    AVGVISIB_85S_060M_201608_EARTH.IMG
    AVGVISIB_85S_060M_201608_EARTH.LBL
  terrain/
    LDSM_80S_80MPP_ADJ.TIF
    LDRM_80S_50MPP_ADJ_ROUGH_100M.TIF
  psr/
    LPSR_80S_20MPP_ADJ.SHP   (plus .SHX, .DBF, .PRJ)
  geology/
    Unified_Geologic_Map_of_the_Moon_GIS/Lunar_GIS/Shapefiles/GeoUnits.shp   (plus companion files)
  craters/
    goransalamuniccar_mooncraters.zip
```

| Data | Source |
| --- | --- |
| DEM, slope, roughness, permanently shadowed regions | [NASA PGDA 90](https://pgda.gsfc.nasa.gov/products/90) |
| Geological units | [USGS Unified Geologic Map of the Moon GIS v2](https://asc-astropedia.s3.us-west-2.amazonaws.com/Moon/Geology/Unified_Geologic_Map_of_the_Moon_GIS_v2.zip) |
| Crater catalogue | [USGS Moon Crater Database v1](https://astrogeology.usgs.gov/search/map/moon_crater_database_v1_salamuni_car) |
| Sun and Earth visibility baselines | [MIT lunar illumination products](https://imbrium.mit.edu/EXTRAS/ILLUMINATION/IMG/) |
| Sun and Earth positions | [NASA JPL Horizons API](https://ssd.jpl.nasa.gov/api/horizons.api) (fetched by `scripts.fetch_sky_table`) |

Descriptions and file-level details are in `backend/data/DATASETS.md`. The optional 5 m site DEMs ([NASA PGDA 78](https://pgda.gsfc.nasa.gov/products/78)) are used only for horizon validation, not for starting the API.

## Deployment

The backend is set up for Docker and Railway, and the frontend for Vercel.

**Backend (Docker, Railway).** Build from `backend/`:

```sh
docker build -t lunar-backend .
```

The image excludes `data/` and `artifacts/`. Its entrypoint runs `scripts/download_data.py` and starts Uvicorn with `PORT` and `WEB_CONCURRENCY`; it does not generate the artifacts. On Railway:

1. Deploy the backend as a service. Add a PostgreSQL service if you want persistent analyses.
2. Add a persistent volume at `/app/data` and put the source files and the generated artifacts there.
3. Set the variables (use your actual PostgreSQL service name; omit `DATABASE_URL` when not using PostgreSQL):

   ```dotenv
   DATA_DIR=/app/data
   ARTIFACTS_DIR=/app/data/artifacts
   DATABASE_URL=${{Postgres.DATABASE_URL}}
   WEB_CONCURRENCY=1
   ```

4. Start or redeploy, then check `/health` and `/docs` on the generated domain.

After a change to source data, artifacts or calculation code, earlier analysis snapshots may return `ANALYSIS_VERSION_CHANGED`. Create a new analysis.

**Frontend (Vercel).** The build uses `npm run build` and the output directory `dist` (`frontend/vercel.json`). Set `VITE_API_BASE_URL` to the backend URL in the project's environment variables.

## Not implemented and known limits

Not implemented (planned):

- Checking Earth contact against Deep Space Network station visibility.
- Using 5 m terrain wherever it exists. The 5 m DEMs for four sites are used for validation only.
- Dates beyond 2028.
- Boulder and temperature data for crew safety.
- Objective scores for `ANCIENT_SUN`, `UNIQUE_OBSERVATION`, `EXPERIMENTAL_SCIENCE`, `EXPLORATION_RISKS` and `GENERAL`. These objectives have no objective-specific score, so only four scores are weighted.

Known limits:

- **Date range.** The sky table covers 2026-01-01 00:00 to 2028-12-31 23:00 UTC. The stay and the landing-date search window must fit inside it.
- **Terrain resolution.** The analysis uses the 80 m DEM. Terrain finer than 80 m and single boulders are not modeled. The 80 m slope data can smooth smaller-scale terrain. Observer height is limited to 1, 5 or 10 m.
- **Earth visibility margin.** The 0.25° margin is provisional. In a 100-location comparison at four sites, the 80 m horizon stayed within about 1.3° of a 5 m hybrid horizon at the 95th percentile. A 45-location LROC stereo check gave 1.532° at 1 m height, with 25.4% of comparisons above 0.25°. Both samples are geographically clustered, so neither establishes a global bound.
- **Baseline comparison.** The NASA/MIT average maps and our results cover different years and resolutions. The correlations of 0.984 and 0.980 should be read with that in mind.
- **Data and artifacts.** Source data and generated artifacts are not in Git. They must be provided separately to each environment, and the container does not generate them.
- **Workers and storage.** Keep `WEB_CONCURRENCY=1`. Without `DATABASE_URL`, analyses are held in memory and lost on restart.

## Data sources and notice

Terrain, shadowed-region and Sun/Earth position data come from NASA (LRO LOLA via PGDA, JPL Horizons). Geological and crater data come from USGS. Baseline maps come from MIT. The map uses NASA Moon Trek tiles and LROC WAC imagery, and the cover image is from NASA SVS CGI Moon Kit.
