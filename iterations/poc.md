# PoC — Iteration Documentation

## Executive Summary

### What was added in this iteration

- OCR-based receipt processing system using PaddleOCR, adapted from a research notebook to a production-ready backend service.
- Image persistence for uploaded receipts stored as BYTEA in PostgreSQL.
- Automatic extraction and persistence of receipt data, with balance updates applied to the affected bank accounts.
- Balance dashboard displaying total and per-account funds for the active user.
- REST API built with FastAPI following a layered architecture (routers, services, repositories, schemas, models).
- CRUD operations for users and bank accounts.
- React + Vite + TypeScript SPA with internationalization (i18n) configured for Spanish (Argentina).
- CI/CD pipelines via GitHub Actions for both frontend and backend repositories.
- CodeRabbit integration for automated code review on pull requests.
- Data model diagram (DBML) reflecting the PoC database schema.
- Initial documentation for the TecaTrack ecosystem.

### Decisions made

- **PaddleOCR as the OCR engine**: selected after a spike evaluating multiple libraries. PaddleOCR showed the best performance on Argentine bank receipt images and handles special characters adequately.
- **BYTEA for image storage**: chosen for simplicity (KISS principle) within the PoC scope. Migration to blob storage (e.g., S3) is flagged as a future improvement.
- **No authentication in the PoC**: the active user is determined by an environment variable in the frontend, reducing complexity to keep the focus on core OCR and balance functionality.
- **Layered backend architecture**: routers, services, repositories, schemas, and models are strictly separated to maintain single responsibility and enable atomic ticket development.
- **OCR engine as singleton**: the PaddleOCR engine initializes once at backend startup to avoid per-request cold starts. Processing runs in an async thread pool (`asyncio.to_thread`) to avoid blocking the API event loop.
- **uv for Python dependency management**: chosen for speed and compatibility with the modern Python ecosystem.
- **Ruff as linter and formatter**: minimal configuration, fast execution, well-suited for FastAPI projects.
- **i18n from day one**: although only Spanish (Argentina) is active, the system was structured to support additional languages without a future refactor.
- **Mock data switch mechanism**: a toggle between mocked and real backend data was implemented to allow frontend development to proceed independently of backend readiness.

---

## User Stories

### Display Totals

**Actor:** User (hardcoded via env var)  
**Functionality:** View the current total balance across all owned accounts on the dashboard.  
**Value:** The user can see how much money they have available across all their bank accounts at a glance.  
**Acceptance criteria:**
- On `DashboardPage` load, a GET request is made to the accounts endpoint for the dummy user.
- The consolidated total and per-account breakdown are displayed.

---

### Update Totals

**Actor:** User  
**Functionality:** See updated account balances immediately after uploading a receipt.  
**Value:** Provides instant feedback confirming that the receipt was processed and balances were adjusted correctly.  
**Acceptance criteria:**
- After a successful receipt upload, the balance screen refreshes and shows the updated totals for both the source and destination accounts.

---

### Upload Receipt

**Actor:** User  
**Functionality:** Upload a bank transfer receipt image from local storage to trigger balance updates.  
**Value:** Allows the user to register a transfer without manual data entry, relying on OCR for extraction.  
**Acceptance criteria:**
- The upload button allows selecting a file from local storage.
- A loading indicator is shown while the receipt is being processed.

---

## Technical Tasks

### Process Receipt

**Actor:** System (backend)  
**Functionality:** Adapt the PaddleOCR notebook code into the backend to process uploaded receipt images via a dedicated endpoint.  
**Value:** Enables the system to automatically extract structured data from receipt images without manual input.  
**Acceptance criteria:**
- Controller that receives a receipt image upload and forwards it for processing.
- Service that takes the receipt and returns a JSON of extracted fields.
- Infrastructure module encapsulating the OCR processing logic.

---

### Bank Account CR

**Actor:** System (backend)  
**Functionality:** Expose `/accounts` endpoints with GET and POST methods to read and create bank account records.  
**Value:** Allows the system to manage users' bank accounts, which are the target of balance updates after receipt processing.  
**Acceptance criteria:**
- Create a bank account with: bank name, balance, CBU, owner name.
- Read bank account data: bank name, balance, CBU, owner name.
- Account router, schemas, service layer, and repository layer implemented.

---

### Receipt Upload Screen Layout

**Actor:** User  
**Functionality:** Modal UI for uploading a receipt, accessible from the main dashboard.  
**Value:** Provides the entry point for the core user flow: uploading a receipt to update account balances.  
**Acceptance criteria:**
- Clicking "Cargar comprobante" on the dashboard opens a modal overlay with a light background dimming effect.

---

### Add Internationalization

**Actor:** Development team  
**Functionality:** Migrate all hardcoded strings in React components to an internationalization system configured for Spanish (Argentina).  
**Value:** Prepares the application to support multiple languages in the future without a full refactor.  
**Acceptance criteria:**
- An i18n library is installed and initialized.
- Spanish (Argentina) is set as the default locale.
- All previously hardcoded UI strings are replaced with i18n keys.

---

### Generate PoC Slides

**Actor:** Development team  
**Functionality:** Create the PoC presentation slides for the academic delivery.  
**Value:** Communicates the project's value proposition, scope, risks, and technology stack to a non-technical audience.  
**Acceptance criteria:**
- Slides include: project name, brand image, motivation, target actors, tentative license, scope, risks, and technologies.
- A live demo of the PoC is shown.

---

### Persist Receipt Image

**Actor:** System (backend)  
**Functionality:** Persist receipt images in PostgreSQL as BYTEA after upload.  
**Value:** Preserves the original receipt for future auditing or data recovery without requiring external storage.  
**Acceptance criteria:**
- The OCR endpoint is updated to persist the received image in PostgreSQL as BYTEA before or alongside processing.

---

### Persist Receipt Data

**Actor:** System (backend)  
**Functionality:** Update and persist the balances of accounts affected by a receipt after OCR processing.  
**Value:** Ensures account balances in the database reflect real processed transfers.  
**Acceptance criteria:**
- OCR service is refactored into a receipt service; OCR logic is delegated exclusively to the infrastructure layer.
- After processing, the affected accounts are retrieved and their balances updated and persisted.

---

### Totals Screen Layout

**Actor:** User  
**Functionality:** Design the main screen layout showing total balance and per-account subtotals.  
**Value:** Gives the user a clear, structured view of their financial situation across all accounts.  
**Acceptance criteria:**
- Layout includes: a title, an upload receipt button, a Total card with the consolidated value, and individual cards per bank account.
- Base styles defined.

---

### Define App Name

**Actor:** Development team  
**Functionality:** Define the provisional name of the application.  
**Value:** Establishes the project identity for all documentation, repositories, and branding.  
**Acceptance criteria:**
- Name defined: **TecaTrack**.

---

### Organize Planning and Review Meetings

**Actor:** Development team  
**Functionality:** Establish a meeting cadence for sprint planning and review sessions.  
**Value:** Keeps the team synchronized and provides checkpoints to validate progress and reprioritize if needed.  
**Acceptance criteria:**
- Planning and review meetings scheduled and held each sprint.

---

### Configure Trello

**Actor:** Development team  
**Functionality:** Set up the Trello board as the project management tool.  
**Value:** Centralizes task tracking, sprint state, and backlog visibility for the whole team.  
**Acceptance criteria:**
- Trello board operational with lists and initial tickets created.

---

### Create Frontend Repository

**Actor:** Development team  
**Functionality:** Create the frontend repository.  
**Value:** Enables collaborative frontend development with version control.  
**Acceptance criteria:**
- Repository created and accessible to the team.

---

### Create Documentation Repository

**Actor:** Development team  
**Functionality:** Create the documentation repository.  
**Value:** Centralizes all project documentation for the TecaTrack ecosystem.  
**Acceptance criteria:**
- Repository created and accessible to the team.

---

### Create Backend Repository

**Actor:** Development team  
**Functionality:** Create the backend repository.  
**Value:** Enables collaborative backend development with version control.  
**Acceptance criteria:**
- Repository created and accessible to the team.

---

### Generate Documentation Template

**Actor:** Development team  
**Functionality:** Create a base documentation file in the repository with the required structure.  
**Value:** Provides a consistent starting point for documentation so the team can fill it in incrementally.  
**Acceptance criteria:**
- Base file committed to the repository with the expected structure in place.

---

### Define Code License

**Actor:** Development team  
**Functionality:** Define the code license for the project.  
**Value:** Establishes the legal terms for use and distribution of the TecaTrack codebase.  
**Acceptance criteria:**
- License file added to the repositories.

---

### Mock Data

**Actor:** Developer  
**Functionality:** Implement a mechanism to consume mocked data for fast frontend testing.  
**Value:** Decouples frontend development from backend availability, allowing parallel progress.  
**Acceptance criteria:**
- Mocked data is consumable for quick frontend tests.
- A switch mechanism exists to toggle between mocked and real backend data.

---

### User CRUD

**Actor:** System (backend)  
**Functionality:** Implement create, read, update, and delete operations for the User entity in the backend.  
**Value:** Establishes the base domain entity on which accounts and receipts are articulated.  
**Acceptance criteria:**
- User router, schemas, service layer, and repository layer implemented.
- Database schema model defined.
- Unit and integration tests for the generated logic where applicable.

---

### GitHub Actions Config — Frontend

**Actor:** Development team / CI  
**Functionality:** Set up a GitHub Actions CI pipeline for the frontend repository.  
**Value:** Automatically validates code quality on every push, catching build and lint issues early.  
**Acceptance criteria:**
- Pipeline validates code compilation.
- Pipeline runs format and linter checks.

---

### GitHub Actions Config — Backend

**Actor:** Development team / CI  
**Functionality:** Set up a GitHub Actions CI pipeline for the backend repository.  
**Value:** Automatically validates code quality on every push, catching build and lint issues early.  
**Acceptance criteria:**
- Pipeline validates compilation with uv.
- Pipeline runs format and linter checks with ruff.

---

### OCR Spike

**Actor:** Development team  
**Functionality:** Research and evaluate OCR libraries and techniques for extracting data from a single bank receipt. Assess the impact of image vs. PDF input.  
**Value:** Reduces technical risk by identifying the most suitable OCR approach before committing to an implementation.  
**Acceptance criteria:**
- A library and technique selected.
- Image vs. PDF tradeoff evaluated and documented.

---

### Configure CodeRabbit

**Actor:** Development team  
**Functionality:** Configure CodeRabbit for automated code review on pull requests.  
**Value:** Improves code quality by providing automated feedback on every PR without blocking the review process.  
**Acceptance criteria:**
- CodeRabbit active and reviewing pull requests in the repositories.

---

### Generate PoC Data Model

**Actor:** Development team  
**Functionality:** Generate a DBML diagram reflecting the database table relationships for the PoC.  
**Value:** Documents the data structure of the system in a visual, exportable format for review and onboarding.  
**Acceptance criteria:**
- DBML file generated with table relationships.
- Diagram reflects only the tables required for the PoC.

---

### Initialize Project

**Actor:** Developer  
**Functionality:** Initialize the backend project with uv, base libraries, and ruff configuration.  
**Value:** Establishes the technical foundation of the backend so the team can start developing consistently.  
**Acceptance criteria:**
- Project initialized with uv.
- Lock file generated with base libraries: fastapi, uvicorn, sqlalchemy, pydantic.
- Ruff configured.
- A functional FastAPI app with a `/health` endpoint running.

---

### AGENT.md — Backend

**Actor:** Developer  
**Functionality:** Create an AGENT.md file in the backend repository.  
**Value:** Provides AI coding agents with the context, conventions, and boundaries of the backend project.  
**Acceptance criteria:**
- AGENT.md file present and committed in the backend repository.

---

### AGENT.md — Frontend

**Actor:** Developer  
**Functionality:** Create an AGENT.md file in the frontend repository.  
**Value:** Provides AI coding agents with the context, conventions, and boundaries of the frontend project.  
**Acceptance criteria:**
- AGENT.md file present and committed in the frontend repository.

---

### Create App Icon

**Actor:** Development team  
**Functionality:** Create the application icon for TecaTrack.  
**Value:** Establishes the visual identity of the application.  
**Acceptance criteria:**
- Icon created and integrated into the project.
