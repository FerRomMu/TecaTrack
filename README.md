# TecaTrack Documentation

<p align="center">
  <img src="assets/app-icon.png" alt="TecaTrack Logo" width="350"/>
</p>

**Introduction**  
TecaTrack is an application for managing receipts and financial transactions using OCR (Optical Character Recognition) technology. It automatically classifies receipt images by bank, extracts structured data using bank-specific processors or an LLM-based fallback, and maps the results to the authenticated user's accounts. Access is secured with Google OAuth 2.0, and users can organize their movements with personal categories. It also supports manual transaction entry and recurring income scheduling.

---

### Repositories

This repository is the documentation hub for the TecaTrack ecosystem. Source code lives in the frontend/backend repos.

- [TecaTrack (Main)](https://github.com/FerRomMu/TecaTrack): Central wiki, documentation, and technical reports.
- [TecaTrack-Frontend](https://github.com/FerRomMu/TecaTrack-Frontend): User interface, components, and API integration.
- [TecaTrack-Backend](https://github.com/FerRomMu/TecaTrack-Backend): REST API, business logic, OCR integration, and database interaction.

---

## Features

- **Authentication**: Sign in with Google (OAuth 2.0). New users complete a one-time registration by providing a valid CUIL. Sessions are backed by JWT access tokens.
- **Receipt Upload with Review**: Upload images of receipts and automatically extract structured data via OCR. The extracted readout is presented for review and editing before anything is persisted.
- **Two-Phase Confirmation**: A receipt stays in `WAITING_CONFIRMATION` until the user explicitly confirms; the transaction is created and balances updated only after confirmation.
- **Multi-bank OCR**: Bank-specific processors for Brubank and Lemon, with an LLM-based (Gemini) fallback for other institutions, routed by a CNN bank classifier.
- **Receipt Mapping**: Extracted data is associated with the user's accounts and balances are updated accordingly.
- **Balance Dashboard**: Visualization of total and per-account balances.
- **Transaction Management**: Dedicated transactions view to browse history, filter by date range and category, and edit existing records.
- **Categories**: Create personal categories with a name and icon, assign them to transactions and recurring incomes, and filter by them.
- **Manual Transaction Entry**: Log income, expenses, or transfers without uploading a receipt.
- **Recurring Income**: Schedule periodic income entries that are registered automatically by a background worker, and view, edit, and filter them in a dedicated section.
- **Timezone-Aware Dates**: Transactions are stored in UTC and shown in the user's local timezone; future dates are rejected by the backend.
- **Account Management**: Create and switch between multiple bank accounts.

---

## Premises and Requirements

- **Authentication**: Access requires signing in with Google (OAuth 2.0). The backend issues stateless JWT access tokens, and every data endpoint resolves the active user from the bearer token.
- **User Identity**: Users must have a valid and unique Argentine CUIL number, captured during the first-time registration flow.
- **Environment Variables (Backend)**:
  - `GEMINI_API_KEY` — required for LLM-based receipt extraction (Default processor).
  - `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_OAUTH_REDIRECT_URI` — Google OAuth 2.0 credentials.
  - `FRONTEND_URL` — used to redirect back to the frontend after the OAuth callback.
  - `JWT_SECRET` — signing key for access/pending tokens (must be at least 32 characters).

---

## Technologies Used

| Component      | Technology              | Reason                                                            |
| -------------- | ----------------------- | ----------------------------------------------------------------- |
| Frontend       | React, Vite, TypeScript | Dynamic interface development and strict typing                   |
| Frontend UI    | Ant Design (AntD)       | Rapid styling and component creation                              |
| Frontend       | React Router DOM        | Client-side routing across app pages                              |
| Frontend       | Axios                   | Centralized HTTP client with typed error handling                 |
| Backend        | Python, FastAPI         | Robust business logic and rapid API creation                      |
| Auth           | Google OAuth 2.0 + JWT  | Delegated sign-in and stateless session tokens                    |
| Database       | PostgreSQL, Alembic     | Relational data persistence and secure migrations                 |
| Infrastructure | PaddleOCR               | Optical Character Recognition (OCR) to extract data from receipts |
| Infrastructure | EfficientNet-B0 (CNN)   | Identifies the originating bank from a receipt image              |
| Infrastructure | Google Gemini (LLM)     | LLM-based receipt data extraction as a fallback processor         |

---

## Domain Vocabulary

### Main Entities

- **User**: Identified by email and a unique CUIL. Authenticated via Google. Can own multiple accounts, receipts, and categories.
- **Account**: Bank account linked to a user. Stores bank name, balance, and CBU. Tracks transactions.
- **Category**: User-owned label with a name and an icon (from a fixed icon set). Optionally assigned to transactions and recurring incomes for organization and filtering.
- **File**: Binary representation of the uploaded receipt image (`BYTEA` in PostgreSQL).
- **Receipt**: Processed receipt linking user and file. Tracks its OCR lifecycle status (`PENDING`, `WAITING_CONFIRMATION`, `PROCESSED`, `FAILED`), stores the raw OCR text and the structured `extracted_data` (JSONB), carries an `is_income` flag, and records `confirmed_at` once the user confirms.
- **Transaction**: Monetary movement linking sender, receiver, source/destination accounts, and optionally a receipt, a category, and a recurring income. Records timezone-aware date fields (`transaction_date`, `transaction_date_source`, `uploaded_at`, `user_timezone`).
- **RecurringIncome**: Scheduled income entry that triggers automatic transaction registration on a defined period (weekly, biweekly, or monthly). Carries an optional description and category.

## Database schema

![Database schema](./assets/db-schema.png)

---

## Upload Receipt Workflow

1. The user signs in with Google; subsequent requests carry a JWT bearer token that identifies the active user.
2. User uploads a receipt image.
3. The CNN classifier identifies the originating bank.
4. The matching bank-specific OCR processor (Brubank, Lemon, or the LLM-based Default) extracts structured data.
5. The File and Receipt are persisted, the receipt moves to `WAITING_CONFIRMATION`, and an OCR readout (with suggested accounts) is returned for review — no transaction is created yet.
6. The user reviews and edits the extracted fields and confirms.
7. On confirmation, the system creates the transaction, updates the affected account balances, and marks the receipt `PROCESSED`.

### Architecture Diagram

![Architecture Diagram](./assets/app_architecture_diagram.png)

### Responsabilities:

- AppNavBar.tsx
  Top-level navigation component that renders the tab bar and routes the user between the main pages of the application (Dashboard, Upload, User).

- UploadPage.tsx
  Page component that hosts the upload section. Renders the tab layout for receipt upload and recurring income, and conditionally mounts UploadReceiptModal when the user triggers the upload
  action.

- UploadReceiptModal.tsx
  Modal component that owns the receipt upload interaction. Holds local state for the selected file and the isIncome flag, calls AccountsService.uploadReceipt() on confirmation, and reports
  success or error back to the user.

- AccountsService (accounts-service.ts)
  HTTP service layer for all account-related operations. Builds the multipart/form-data payload and delegates the actual HTTP call to apiClient. Also handles account creation and balance.

- apiClient.ts (Axios — third-party)
  Centralized Axios instance that is the single HTTP exit point of the frontend. Configured with the base URL and a response interceptor that maps backend error codes to typed ApiError objects with i18n translation.

- get_current_user (core/dependencies.py)
  FastAPI dependency that validates the JWT bearer token, ensures it is an access token, and resolves the authenticated User. Injected into every protected router so each use case operates on the identity carried by the token.

- receipt_router.py — POST /receipts/upload-receipt and POST /receipts/{receipt_id}/confirm
  FastAPI router that receives the upload request (file and is_income flag) and the later confirmation request. Resolves the active user via get_current_user, resolves service dependencies via injection, and delegates both phases to ReceiptService.

- ReceiptService (receipt_service.py)
  Central orchestrator of the two-phase upload use case. On upload it calls each infrastructure component in sequence — image conversion, bank classification, OCR extraction — persists the File and Receipt, leaves the receipt in WAITING_CONFIRMATION, and returns an OCR readout. On confirm it validates the edited data, creates the transaction, updates balances, and marks the receipt PROCESSED. Guarantees that File and Receipt are persisted even if extraction fails.

- TransactionService (transaction_service.py)
  Handles the business logic for creating a transaction record and applying the resulting balance delta to the source and destination accounts. Delegates persistence to TransactionRepository.

- AccountService (account_service.py)
  Resolves and validates the accounts involved in a transaction by matching the user's CUIL with the CBU and bank name extracted from the receipt. Provides concurrency-safe balance update operations.

- ReceiptProcessor (receipt_classifier.py)
  Selects and executes the appropriate bank-specific parsing strategy after the classifier determines the bank. Applies the processor's rules to the raw OCR output and returns a structured OCRResponse.

- GeminiClient (gemini_client.py)
  Implementation of the LLMClient interface that sends a prompt to the Google Gemini API. Used exclusively by the Default processor as a fallback when no bank-specific processor matches the classified receipt.

- ReceiptClassifier (receipt_classifier.py)
  Preprocesses the raw image bytes and passes them to ClassifierEngine to obtain a bank prediction. Returns the corresponding ReceiptProcessor instance to use for structured data extraction.

- OCRProcessor (ocr_processor.py)
  Drives the raw text extraction using OCREngine and forwards the result to the active ReceiptProcessor for structured parsing. Returns both the structured fields and the raw OCR text for storage.

- ClassifierEngine — EfficientNet-B0 fine-tuned (third-party)
  Thread-safe lazy singleton that loads and caches the fine-tuned EfficientNet-B0 model in memory. Exposes get() returning the model and class map, ensuring the model is initialized only once across all requests.

- OCREngine — PaddleOCR (third-party)
  Thread-safe lazy singleton wrapping PaddleOCR, configured with Spanish language and text-orientation detection. Runs inference in a thread pool via asyncio.to_thread to avoid blocking the async event loop.

- FileRepository (file_repository.py)
  Accesses the files table and persists the raw BYTEA image data. Decoupled from receipt processing so the binary artifact is stored regardless of whether OCR succeeds.

- ReceiptRepository (receipt_repository.py)
  Accesses the receipts table. Persists the receipt record with its OCR status, raw extracted text, and is_income flag. Provides update() to reflect the final processing state after extraction completes.

- TransactionRepository (transaction_repository.py)
  Accesses the transactions table. Persists new transaction records and supports updates to existing ones, keeping monetary movements linked to the accounts and receipts they reference.

- AccountRepository (account_repository.py)
  Accesses the accounts table. Supports account creation, retrieval by CBU, and balance updates used by AccountService during transaction processing.

- transactional_session() / database.py (SQLAlchemy — third-party)
  Async context manager over SQLAlchemy's async_sessionmaker. Wraps all repository operations in a single database transaction with automatic commit on success and rollback on failure. It is the sole point of contact with PostgreSQL.

- Google Gemini API — gemini-3.1-flash-lite (third-party)
  External LLM service invoked by GeminiClient to extract structured fields from receipts that do not match a known bank processor. Receives a text prompt with the raw OCR output and returns a structured JSON response.

- PostgreSQL — receipts, transactions, accounts, users (third-party)
  Relational database that persists all application state. Accessed exclusively through transactional_session() to ensure ACID guarantees across all write operations.

---
## Important Technical Decisions

**Layered Architecture (Backend)**
Structured using `routers`, `services`, `repositories`, `schemas`, and `models` to maintain separation of concerns and allow atomic development.

**Image Persistence (Backend)**
Persisted using `BYTEA` in PostgreSQL following the KISS principle, with potential to migrate to dedicated blob storage later.

**OCR Engine Optimization**
PaddleOCR engine is initialized as a singleton at startup to prevent cold-start delays. Receipt processing runs in an async thread pool (`asyncio.to_thread`) to avoid blocking the API event loop.

**Account Matching**
Accounts are reliably matched by combining the user's CUIL with the CBU and bank name extracted from the receipt.

**CNN-based Bank Classifier**
A convolutional neural network classifies the receipt image to determine the originating bank before OCR processing, enabling routing to the correct processor without manual input.

**Bank-specific Processor Architecture with LLM Fallback**
Each supported bank (Brubank, Lemon) has a dedicated OCR processor with tuned configuration. Unrecognized banks fall through to a Default processor that uses Google Gemini to extract receipt data via an LLM prompt.

**Delegated Authentication with Stateless Tokens**
Sign-in is delegated to Google (OAuth 2.0), avoiding local credential storage. The backend issues stateless JWT access tokens, and new users are onboarded through a short-lived pending token that requires a valid, unique CUIL before the account is created.

**Two-Phase Receipt Confirmation**
OCR can misread a receipt, so extraction and persistence are decoupled: the receipt is stored in `WAITING_CONFIRMATION` with its extracted data, and the transaction is only created after the user reviews, edits, and explicitly confirms.

**Timezone Policy**
Timestamps are stored in UTC; the frontend sends ISO 8601 dates with an explicit offset plus the user's IANA timezone, and the backend (not the client clock) is the authority for rejecting future dates.

**User-Scoped Categories**
Categories belong to individual users and behave identically across transactions and recurring incomes; a movement can only reference a category owned by the same user.

---

## Considerations for Future Development

- Support for receipts from additional financial institutions beyond Brubank and Lemon.
- Voice transaction input for logging transactions without receipts.
- Expense Reservations: setting money aside for upcoming expenses with future-spend visibility.
