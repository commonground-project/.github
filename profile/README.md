# CommonGround

CommonGround is a discussion platform that aims to facilitate concensus through rationale discussions. Users can post viewpoints, reference sources, and engage with opposing perspectives in a structured way, backed by a recommendation system designed to surface dialogue across, rather than within, ideological bubbles.

The platform was supported by [NYCU Software Development Club](https://sdc.nycu.club).

## Repositories

The CommonGround stack is split across the following repositories:

| Repository | Description | Stack |
| --- | --- | --- |
| [`frontend`](https://github.com/commonground-project/frontend) | The web application served at [commonground.tw](https://commonground.tw). | Next.js, pnpm, Playwright |
| [`backend`](https://github.com/commonground-project/backend) | The primary application server handling users, posts, viewpoints, and authentication. | Java, Spring Boot, Gradle |
| [`api`](https://github.com/commonground-project/api) | The OpenAPI specification and Swagger documentation shared between the frontend and backend. | OpenAPI 3, Swagger UI |
| [`recommendation-system`](https://github.com/commonground-project/recommendation-system) | A recommendation worker that ranks viewpoints per user and writes results into a Redis ZSET. The backend reads recommendations from the same Redis instance — the two services do not call each other directly. | Python, Redis, OpenAI |
| [`reference-system`](https://github.com/commonground-project/reference-system) | An offline LLM job that reads issues and viewpoints from the backend's HTTP API, generates RAG / few-shot summaries with citations, and `PUT`s the results back to the backend. | Python, LangChain |

## Self-Hosting

Each service ships with its own Dockerfile, and the application is designed to run as a small set of containers behind the frontend.

### Prerequisites

- Docker and Docker Compose
- A Redis instance (used by `recommendation-system`)
- A PostgreSQL instance (used by `backend`)
- Google OAuth2 credentials (used by `backend` for sign-in)
- An OpenAI API key (used by `recommendation-system` for embeddings)

### Steps

1. **Clone the repositories** you intend to run. At minimum you will need `frontend`, `backend`, `recommendation-system`, and `reference-system`.

   ```bash
   git clone https://github.com/commonground-project/frontend.git
   git clone https://github.com/commonground-project/backend.git
   git clone https://github.com/commonground-project/recommendation-system.git
   git clone https://github.com/commonground-project/reference-system.git
   ```

2. **Configure each service.** Every repository contains a README describing the environment variables it expects. The key items are:
   - `backend`: copy `application.properties` into `application-dev.properties` and fill in your Google OAuth2 client ID and secret.
   - `recommendation-system`: set `BASE_URL`, `BEARER_API_TOKEN`, `OPENAI_API_KEY`, and Redis connection variables. This worker shares its Redis instance with the backend — both must point at the same Redis.
   - `reference-system`: configure its `.env` with credentials for the backend's HTTP API (it reads from and `PUT`s back to the deployed backend) and an OpenAI / LLM key for summarization.
   - `frontend`: copy `example.env` to `.env` and point it at your backend.

3. **Build the images.** Run `docker build` in each service's repository root to produce a local image.

4. **Run the stack.** Start Redis and PostgreSQL, then bring up `backend` and the `frontend`. `recommendation-system` runs as a worker against the shared Redis; `reference-system` runs as a batch job against the backend's HTTP API. Neither is on the request path, so the site is usable without them, you just won't get personalized rankings or AI-generated reference summaries.

5. **Verify the deployment** by visiting the frontend in a browser and signing in via Google OAuth.

For per-service configuration details, refer to the README of each repository linked above.

## Project Status

**This project is archived.** CommonGround is no longer actively maintained, and the repositories under this organization are preserved for reference and for anyone who wishes to fork, study, or build on the work.

Future authors interested in reviving the project, taking ownership of the domain, or asking about the codebase are welcome to get in touch at **contact@commonground.tw**.

