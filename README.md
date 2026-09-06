# InfraRAT

InfraRAT is a research data platform for exploring roughly 20,000 rat-behaviour sessions from McMaster University's Szechtman Lab. It brings experimental metadata and the original FRDR files into one searchable interface, so researchers can move from a cohort-level question to an individual session without stitching the archive together by hand.

This was built by a five-person McMaster capstone team in collaboration with Dr. Henry Szechtman and Dr. Anna Dvorkin-Gheva. For a shorter account of the project and my role, see the [project case study](https://agoodyer.com/projects/infrarat/).

![InfraRAT home page](docs/Extras/UserGuide/homepage.png)

## The research problem

The lab's public dataset spans video, tracking data, derived measurements, and supporting files collected across many experiments. The files are available through the Federated Research Data Repository (FRDR), but the archive alone does not make it easy to answer questions across sessions: which animals received a particular treatment, which apparatus was used, or which files belong to a matching cohort.

InfraRAT adds a normalized PostgreSQL metadata layer and a browser-based research workflow around that archive. It does not replace the source dataset; it helps researchers find and inspect the relevant records, then links them back to the original files.

## What the application supports

- **Structured search:** filter sessions by treatment, regimen, brain manipulation, apparatus, session details, and available file formats.
- **Natural-language search:** ask a question or describe a desired selection. An OpenAI-compatible language model produces PostgreSQL for the matching workflow and returns its rationale alongside the generated query.
- **Dataset inspection:** review session metadata and the associated temporal data before downloading source files.
- **Analysis views:** compare groups with bar charts, line charts, categorical heatmaps, spatial heatmaps, path plots, and velocity profiles.
- **File retrieval:** follow records back to their corresponding files in FRDR.

| Query and inspect | Build visualizations |
| --- | --- |
| ![Toolbox view showing session data and a path plot](docs/Extras/UserGuide/ToolBoxPage.png) | ![InfraRAT visualization choices](docs/Extras/UserGuide/Visualizations.png) |

## How it is built

The React and TypeScript frontend is served by Nginx. It calls a FastAPI backend that owns querying, filtering, visualizations, inventory summaries, and file retrieval. PostgreSQL stores the normalized experimental metadata and is initialized from the data bundled with the backend database image. Docker Compose runs the three services together.

The natural-language tools are provider-agnostic: the backend uses an OpenAI-compatible API URL, key, and model name supplied through environment variables. The generated SQL and its rationale are surfaced as part of the selection response, rather than hiding the database operation behind a chat answer.

## What I worked on

InfraRAT was a team project, so the repository reflects shared work. My contributions included:

- establishing early frontend structure, routing, home and experiment pages, and the first query-interface mockups;
- helping connect the frontend, backend, database, and language-model workflow;
- Dockerizing the system and setting up staging deployment and CodeQL automation;
- adding backend tests and contributing to requirements, design, and verification documentation.

## Run it locally

You will need Docker with Compose support.

```bash
git clone https://github.com/OCD-Rats-Capstone/InfraRAT.git
cd InfraRAT
touch .env
docker compose up -d --build
```

Compose expects a root `.env` file even if you are only using the structured tools. To enable the natural-language features, configure an OpenAI-compatible provider:

```dotenv
OPENAI_API_KEY=your-api-key
LLM_BASE_URL=https://your-provider.example/v1
LLM_MODEL=your-model-name
```

The default Compose override exposes:

| Service | Address |
| --- | --- |
| Web application | <http://localhost> |
| Backend API | <http://localhost:8000> |
| OpenAPI documentation | <http://localhost:8000/docs> |
| PostgreSQL | `localhost:5433` |

Useful lifecycle commands:

```bash
docker compose logs -f
docker compose logs -f backend
docker compose down
docker compose up -d --build
```

`docker compose down -v` also removes the local database volume, so use it only when you want a clean data restore.

## Development checks

Backend tests:

```bash
cd src/ocd-rat-backend
python3 -m pytest tests/ -v --tb=short
```

Frontend checks:

```bash
cd src/ocd-rat-frontend
npm ci
npm run build
npm run lint
```

At the current head, the frontend production build succeeds. The repository still has validation debt: 146 of 150 backend tests pass, with one download-path assertion and three running-service performance checks failing, and ESLint reports 46 errors plus 2 warnings. Those results are documented here rather than implying that every check is green.

## Repository guide

```text
.
├── src/ocd-rat-backend/   FastAPI routes, services, tests, and database image
├── src/ocd-rat-frontend/  React application and Nginx configuration
├── docs/                  Requirements, design, verification, and user documents
├── refs/                  Research and course reference material
└── docker-compose*.yml    Shared, development, and deployment configurations
```

The main supporting documents are the [user guide](docs/Extras/UserGuide/UserGuide.pdf), [software requirements specification](docs/SRS-Volere/SRS.pdf), [module guide](docs/Design/SoftArchitecture/MG.pdf), [module interface specification](docs/Design/SoftDetailedDes/MIS.pdf), and [verification and validation report](docs/VnVReport/VnVReport.pdf).

## Team

Aidan Goodyer, Jeremy Orr, Leo Vugert, Nathan Perry, and Timothy Pokanai, with project partners Dr. Henry Szechtman and Dr. Anna Dvorkin-Gheva.

## Status

InfraRAT was developed during the 2025–2026 McMaster Software Engineering capstone. This repository preserves the completed application and its project documentation. Natural-language search requires provider credentials, and some download workflows depend on the continued availability and structure of the public FRDR archive.

## License

[MIT](LICENSE)
