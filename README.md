# TecaTrack Documentation

<p align="center">
  <img src="assets/app-icon.png" alt="TecaTrack Logo" width="350"/>
</p>

**Introduction**  
TecaTrack is an application for managing receipts and financial transactions using OCR (Optical Character Recognition) technology. It automatically classifies receipt images by bank, extracts structured data with bank-specific processors or an LLM-based fallback, and maps the results to user accounts. The project has grown beyond its initial Proof of Concept: it now supports authenticated multi-user access, multiple banks, category management, manual transaction entry, and recurring income.

---

### Repositories

This repository is the documentation hub for the TecaTrack ecosystem. Source code lives in the frontend/backend repos.

- [TecaTrack (Main)](https://github.com/FerRomMu/TecaTrack): Central wiki, documentation, and technical reports.
- [TecaTrack-Frontend](https://github.com/FerRomMu/TecaTrack-Frontend): User interface, components, and API integration.
- [TecaTrack-Backend](https://github.com/FerRomMu/TecaTrack-Backend): REST API, business logic, OCR integration, and database interaction.

---

## Features

- **Authentication**: Google OAuth 2.0 login with JWT-protected endpoints.
- **Receipt Upload & Confirmation**: Upload a receipt image, review the extracted OCR readout, and confirm before the transaction is persisted.
- **OCR Processing**: Automatic extraction of key data, with bank-specific processors for Brubank and Lemon plus a Gemini LLM fallback for unrecognized banks.
- **Receipt Classifier**: CNN-based image classifier (EfficientNet-B0) that identifies the originating bank from a receipt image.
- **Transaction Management**: Create, list (paginated), edit, and filter transactions linked to user accounts.
- **Manual Transaction Entry**: Log income, expenses, or transfers without uploading a receipt.
- **Account Management**: Create and switch between multiple bank accounts.
- **Categories**: Create categories with icons and assign them to transactions and recurring incomes; filter lists by category.
- **Recurring Income**: Define, edit, and filter recurring income entries processed automatically by a background worker.
- **Balance Dashboard**: Visualization of total and individual account balances.

---

## Premises and Requirements

- **Authentication**: Access requires Google OAuth 2.0 login; the backend issues a JWT and all endpoints are protected. The active user identity is derived from the token, not from request payloads.
- **OCR Scope**: Bank-specific processors for Brubank and Lemon, with a Gemini LLM fallback for any other bank.
- **User Identity**: Users must have a valid Argentine CUIL number.
- **Timezone Handling**: Transactions are stored with the user's local timestamp; future-dated transactions are rejected.

---

## Architecture

The system follows a client-server architecture:

- **Frontend**: Single Page Application (SPA) developed with React, Vite, and TypeScript.
- **Backend**: REST API developed in Python using FastAPI, organized in a layered architecture with a dedicated infrastructure layer for OCR, the receipt classifier, and the LLM client, plus background workers for recurring income.
- **Database**: PostgreSQL for relational data persistence.

### Architecture Diagram

![Architecture Diagram](./assets/app_architecture_diagram.png)

---

## Technologies Used

| Component             | Technology                  | Reason                                                                 |
| --------------------- | --------------------------- | --------------------------------------------------------------------- |
| Frontend              | React, Vite, TypeScript     | Dynamic interface development and strict typing                       |
| Frontend UI           | Ant Design (AntD)           | Rapid styling and component creation                                  |
| Frontend Routing/HTTP | React Router DOM, Axios     | Client-side routing and centralized API communication                |
| Backend               | Python, FastAPI             | Robust business logic and rapid API creation                         |
| Authentication        | Google OAuth 2.0, JWT       | Delegated identity and stateless, protected endpoints                |
| Database              | PostgreSQL, Alembic         | Relational data persistence and secure migrations                    |
| OCR Engine            | PaddleOCR                   | Optical Character Recognition to extract data from receipts          |
| Receipt Classifier    | PyTorch (EfficientNet-B0)   | CNN that identifies the originating bank from a receipt image        |
| LLM Fallback          | Google Gemini               | Data extraction for receipts from banks without a dedicated processor |

---

## Domain Vocabulary

### Main Entities

- **User**: Identified by CUIL. Owns multiple accounts, receipts, and categories.
- **Account**: Bank account linked to a user. Stores bank name, balance, and CBU. Tracks transactions.
- **File**: Binary representation of the uploaded receipt image (`BYTEA` in PostgreSQL).
- **Receipt**: Processed receipt linking user and file. Tracks OCR status (`PENDING`, `WAITING_CONFIRMATION`, `PROCESSED`, `FAILED`), the income flag, extracted data, OCR text, and confirmation timestamp.
- **Category**: User-defined label with an icon, assignable to transactions and recurring incomes for organization and filtering.
- **Transaction**: Monetary movement linking sender, receiver, source/destination accounts, and optionally a receipt, category, and recurring income. Stores a timezone-aware date along with its source (OCR, manual, or corrected).
- **Recurring Income**: Scheduled income entry (amount, frequency, next execution date, target account, optional category) processed automatically by a background worker.

## Database schema

![Database schema](./assets/db-schema.png)

---

## Main System Workflow

1. The user signs in with Google OAuth; the backend issues a JWT used to authenticate all subsequent requests.
2. The user uploads a receipt image (Brubank, Lemon, or another bank).
3. The classifier identifies the originating bank and routes the image to the matching OCR processor, or to the Gemini LLM fallback for unrecognized banks.
4. The system persists the image and returns the extracted OCR readout, leaving the receipt in a `WAITING_CONFIRMATION` state.
5. The user reviews and confirms the readout; the transaction is persisted with a localized timestamp and the affected account balances are updated.
6. The user views updated balances, transactions, and categories on the dashboard.

Manual transactions and recurring income entries follow the same persistence and balance-update logic without the OCR step.

### Component Responsibilities (receipt upload flow)

- AppNavBar.tsx
  Top-level navigation component that renders the tab bar and routes the user between the main pages of the application (Dashboard, Transactions, Categories, Schedule, Upload, User).

- UploadPage.tsx
  Page component that hosts the upload section. Renders the tab layout for receipt upload and recurring income, and conditionally mounts UploadReceiptModal when the user triggers the upload
  action.

- UploadReceiptModal.tsx
  Modal component that owns the receipt upload interaction. Sends the selected file to the backend, then renders ReceiptConfirmForm with the returned OCR readout so the user can review the data and pick the origin/destination accounts before confirming. Reports success or error back to the user.

- ReceiptConfirmForm.tsx
  Form that presents the extracted OCR fields for review and correction. Replaces the previous is-income checkbox with from/to account dropdowns, disables future dates, and calls confirmReceipt() to persist the transaction.

- AccountsService (accounts-service.ts)
  HTTP service layer for all account-related operations. Builds the multipart/form-data payload for uploadReceipt() (which returns the OCR readout) and exposes confirmReceipt() for the second phase. Delegates the actual HTTP call to apiClient and also handles account creation and balances.

- apiClient.ts (Axios — third-party)
  Centralized Axios instance that is the single HTTP exit point of the frontend. Attaches the JWT bearer token, and a response interceptor maps backend error codes to typed ApiError objects with i18n translation.

- receipt_router.py — POST /receipts/upload-receipt and POST /receipts/{id}/confirm
  FastAPI router for the two-phase flow. The identity is derived from the authenticated user's JWT (no user_id in the request body); upload receives the file and returns the OCR readout, while confirm receives the reviewed data. Resolves service dependencies via injection and delegates the use case to ReceiptService.

- ReceiptService (receipt_service.py)
  Orchestrator of the receipt use case, split into process_upload and confirm_receipt. process_upload runs image conversion, bank classification, and OCR extraction, then persists the File and Receipt (in a WAITING_CONFIRMATION state) and returns the readout. confirm_receipt validates the reviewed data, creates the transaction, and delegates balance updates to the domain services. Guarantees that File and Receipt are persisted even if extraction fails.

- TransactionService (transaction_service.py)
  Handles the business logic for creating a transaction record and applying the resulting balance delta to the source and destination accounts. Enforces future-date validation and localizes timestamps to the user's timezone. Delegates persistence to TransactionRepository.

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
  Accesses the receipts table. Persists the receipt record with its OCR status, raw extracted text, is_income flag, and confirmation timestamp. Provides update() to reflect the final processing state after extraction and confirmation complete.

- TransactionRepository (transaction_repository.py)
  Accesses the transactions table. Persists new transaction records and supports updates to existing ones, keeping monetary movements linked to the accounts and receipts they reference.

- AccountRepository (account_repository.py)
  Accesses the accounts table. Supports account creation, retrieval by CBU, and balance updates used by AccountService during transaction processing.

- transactional_session() / database.py (SQLAlchemy — third-party)
  Async context manager over SQLAlchemy's async_sessionmaker. Wraps all repository operations in a single database transaction with automatic commit on success and rollback on failure. It is the sole point of contact with PostgreSQL.

- Google Gemini API — gemini-3.1-flash-lite (third-party)
  External LLM service invoked by GeminiClient to extract structured fields from receipts that do not match a known bank processor. Receives a text prompt with the raw OCR output and returns a structured JSON response.

- PostgreSQL — users, accounts, categories, files, receipts, transactions, recurring_incomes (third-party)
  Relational database that persists all application state. Accessed exclusively through transactional_session() to ensure ACID guarantees across all write operations.

---
## Important Technical Decisions

**Layered Architecture (Backend)**
Structured using `routers`, `services`, `repositories`, `schemas`, and `models`, with an `infrastructure` layer for OCR, the classifier, and the LLM client, to maintain separation of concerns and allow atomic development.

**Image Persistence (Backend)**
Persisted using `BYTEA` in PostgreSQL following the KISS principle for the current scope, with potential to migrate to dedicated blob storage later.

**OCR Engine Optimization**
The PaddleOCR engine is initialized as a singleton at startup to prevent cold-start delays. Receipt processing runs in an async thread pool (`asyncio.to_thread`) to avoid blocking the API event loop.

**Receipt Classifier (CNN)**
An EfficientNet-B0 classifier identifies the originating bank (Brubank, Lemon, or unknown) from the receipt image and routes processing to the appropriate OCR processor.

**LLM Fallback**
A Gemini-based Default processor extracts data from receipts whose bank has no dedicated processor, providing graceful coverage for unknown institutions.

**Two-Phase Receipt Flow**
Upload extracts and returns the OCR readout in a `WAITING_CONFIRMATION` state so the user can review and correct ambiguous extractions before the transaction is persisted.

**Authentication**
Google OAuth 2.0 with JWT issuance. Identity is derived from the token rather than from request payloads, and all endpoints are protected.

**Timezone-Aware Transactions**
Timestamps are stored in the user's local timezone, with the date source recorded (OCR, manual, or corrected). Future-dated transactions are rejected at creation.

**Account Matching**
Accounts are reliably matched by combining the user's CUIL with the CBU and bank name extracted from the receipt.

---

## Considerations for Future Development

- Voice transaction input for logging transactions without receipts.
- Expense Reservations: setting money aside for upcoming expenses with future-spend visibility.
- Additional bank-specific OCR processors beyond Brubank and Lemon.
- Migration to dedicated blob storage (e.g., S3) for files.
- Multi-language support (currently Spanish/Argentina only).
