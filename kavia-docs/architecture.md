# OTT Fan Engagement Platform – Architecture Documentation

## Overview

The OTT Fan Engagement Platform is a modular, end-to-end web application designed to enhance the fan experience for OTT sports platforms. It achieves this by enabling fans to interact in real time through features such as live chat, polls, content sharing, and personalized user profiles. The platform is composed of multiple independently deployable containers/services, each responsible for specific functionality.

**Current Core Containers:**
- `main_database` : MongoDB instance for persistent data storage.
- `web_frontend` : React-based web UI for user interaction.

Future plans include the addition of a backend API container to further separate concerns between data storage and client functionality.

---

## Container Structure & Components

### 1. Main Database (`main_database`)

- **Type:** Database Service
- **Technology:** MongoDB
- **Primary Role:** Persistence for user profiles, session data, engagement content, event data, polls, and live activity.
- **Accessible via:** (Planned) Backend API and other internal service REST API endpoints.
- **Initialization:** Managed via shell scripting (`startup.sh`) for user creation, DB bootstrap, and environment variable prep.
- **Internal Utility:** Includes a `db_visualizer` mini-service that runs as a web server (Node.js/Express) providing a database viewer UI and REST endpoints for direct database operations, primarily for dev/test/operations.
  - Supports Postgres, MySQL, SQLite, and MongoDB for flexible development; in production, MongoDB is the primary driver.
- **Notable Endpoints (from `server.js`):**
  - `GET /api/databases` — List available/connected database types.
  - `GET /api/:db/tables` — List tables/collections in the specified DB.
  - `GET /api/:db/tables/:table/data` — Access data from a specific table/collection (with limit param).

#### Environment/Secrets Handling

- Inherits variables from `.env`-style files for DB connection details, e.g.:
  - `MONGODB_URL`, `MONGODB_DB`
- Credentials and users (including admin/app user) provisioned on first launch.
- Local connection string management via `db_connection.txt` and `.env` files for utility access.

---

### 2. Web Frontend (`web_frontend`)

- **Type:** Frontend (SPA)
- **Technology:** React (vanilla, modern CSS, no heavy UI libraries)
- **Primary Role:** User-facing application; allows fans to log in, join discussions, participate in live polls/quizzes, manage profiles, and view shared content.
- **Dependencies:** Will connect to the backend API service (future), which will in turn interact with the database. In current dev/test setups, it may be configured to hit the `db_visualizer` endpoints for direct data interactions.
- **Features (from container descriptor):**
  - Live chat and discussions
  - Real-time polls and quizzes
  - User authentication/profile management
  - Content sharing
  - Event scheduling, notifications
  - Gamified leaderboards
- **UI/UX:** Modern, minimalist, and responsive design with theme-toggle (light/dark mode).

---

## Planned Backend API (Not yet implemented)

- **Type:** Backend/API service
- **Primary Role:** Serve as the intermediary between the frontend and the main database, providing RESTful API endpoints for all application data, encapsulating business logic and validation, and enforcing security/authentication.
- **Interfaces:** RESTful JSON APIs.
- **Dependencies:** Talks to `main_database`, serves requests from `web_frontend`.

---

## Inter-Container Interfaces & Data Flow

At a high level, the data and control flows as follows:

1. The React frontend presents UI components and collects user input.
2. For all data requests (user data, content feeds, poll results, etc.):
    - **Current State:** Can hit REST endpoints exposed by the db_visualizer (for admin/utility tasks) or go directly via MongoDB drivers in a development mode.
    - **Planned/Prod State:** Will interact via the dedicated backend API service.
3. The backend (planned) processes logic, personas, authorization, and queries the database. 
4. MongoDB persists and retrieves documents as required.

---

## Architecture Diagram

```mermaid
flowchart TD
    subgraph Frontend
      A[User Browser] --> B[web_frontend<br/>(React SPA)]
    end
    subgraph Backend
      C[Backend API Service<br/>(planned)]
    end
    subgraph Database
      D[main_database<br/>(MongoDB)]
      E[db_visualizer<br/>(Node+Express)]
    end

    B -- REST API Calls (Planned) --> C
    C -- CRUD/Query/Session --> D
    E -- Utility Endpoints/API --> D
    B -- (Dev/Test/Utility) Direct REST --> E
```

---

## Container Dependencies

- **web_frontend:** 
  - *Development/testing:* Can be configured to connect directly to db_visualizer for data visualization/CRUD.
  - *Production/target:* Will only connect to Backend API.
- **main_database:** 
  - No direct dependencies on other containers; receives API calls.
- **db_visualizer:** 
  - Internal/utility, only depends on MongoDB connection.

---

## Summary Table

| Container          | Platform/Tech           | Key Role                                    | Interfaces                     | Dependencies         |
|--------------------|------------------------|----------------------------------------------|---------------------------------|----------------------|
| main_database      | MongoDB (container)    | Store persistent user & engagement data      | Mongo CLI, REST API (planned)  | web_frontend (via backend) |
| db_visualizer      | Node.js/Express        | Utility web viewer/admin API                 | REST/HTTP for admin ops         | main_database        |
| web_frontend       | React                  | Fan-facing UI web application                | HTTP, REST (to backend/API)     | (planned) backend API |
| backend_api (plan) | Node/Express, Python?  | App/IF logic, API routing, authorization     | RESTful API (JSON)              | main_database        |

---

## Directory Structure (Relevant)

- `fan-engagement-platform-025483f1/main_database/` …. MongoDB DB, helper scripts, `db_visualizer` subfolder (UI/API utility)
- `fan-engagement-platform-fe5af5ef/web_frontend/` …. React SPA, assets, branding, theming

---

## Planned Extension & Scalability

The architecture allows horizontal scaling (k8s/docker), clear separation of concerns, and easy extension for additional services (notification, analytics, gamification microservices, etc.) as OTT engagement grows.

---

## Appendix: Key Files

- `main_database/startup.sh` – setup & bootstrapping logic for DB container.
- `main_database/db_visualizer/server.js` – REST endpoint server for DB utility access.
- `web_frontend/src/App.js` – main React SPA UI logic.

---

Task completed: Comprehensive architecture documentation—including a mermaid diagram—has been generated for the OTT fan engagement platform.
