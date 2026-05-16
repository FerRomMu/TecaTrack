# TecaTrack Documentation

<p align="center">
  <img src="assets/app-icon.png" alt="TecaTrack Logo" width="350"/>
</p>

**Introduction**  
TecaTrack is an application for managing receipts and financial transactions using OCR (Optical Character Recognition) technology. It automatically classifies receipt images by bank, extracts structured data using bank-specific processors or an LLM-based fallback, and maps the results to user accounts. It also supports manual transaction entry and recurring income scheduling.

---

### Repositories

This repository is the documentation hub for the TecaTrack ecosystem. Source code lives in the frontend/backend repos.

- [TecaTrack (Main)](https://github.com/FerRomMu/TecaTrack): Central wiki, documentation, and technical reports.
- [TecaTrack-Frontend](https://github.com/FerRomMu/TecaTrack-Frontend): User interface, components, and API integration.
- [TecaTrack-Backend](https://github.com/FerRomMu/TecaTrack-Backend): REST API, business logic, OCR integration, and database interaction.

---

## Features

- **Receipt Upload**: Upload images of receipts and automatically extract structured data via OCR.
- **Multi-bank OCR**: Bank-specific processors for Brubank and Lemon, with an LLM-based (Gemini) fallback for other institutions.
- **Receipt Mapping**: Extracted data is associated with the user's accounts and balances are updated accordingly.
- **Balance Dashboard**: Visualization of total and per-account balances.
- **Transaction Management**: View recent transactions, filter by date range, and edit existing records.
- **Manual Transaction Entry**: Log income, expenses, or transfers without uploading a receipt.
- **Recurring Income**: Schedule periodic income entries that are registered automatically by a background worker.
- **Account Management**: Create and switch between multiple bank accounts.

---

## Premises and Requirements

- **No Authentication** (Temporarily implemented): The active user is determined by an environment variable in the frontend. A dev-only endpoint allows switching between users during testing.
- **User Identity**: Users must have a valid Argentine CUIL number.
- **Environment Variables**: `GEMINI_API_KEY` is required in the backend for LLM-based receipt extraction (Default processor).

---

## Technologies Used

| Component      | Technology              | Reason                                                            |
| -------------- | ----------------------- | ----------------------------------------------------------------- |
| Frontend       | React, Vite, TypeScript | Dynamic interface development and strict typing                   |
| Frontend UI    | Ant Design (AntD)       | Rapid styling and component creation                              |
| Frontend       | React Router DOM        | Client-side routing across app pages                              |
| Backend        | Python, FastAPI         | Robust business logic and rapid API creation                      |
| Database       | PostgreSQL, Alembic     | Relational data persistence and secure migrations                 |
| Infrastructure | PaddleOCR               | Optical Character Recognition (OCR) to extract data from receipts |
| Infrastructure | CNN classifier          | Identifies the originating bank from a receipt image              |
| Infrastructure | Google Gemini (LLM)     | LLM-based receipt data extraction as a fallback processor         |

---

## Domain Vocabulary

### Main Entities

- **User**: Identified by CUIL. Can own multiple accounts and receipts.
- **Account**: Bank account linked to a user. Stores bank name, balance, and CBU. Tracks transactions.
- **File**: Binary representation of the uploaded receipt image (`BYTEA` in PostgreSQL).
- **Receipt**: Processed receipt linking user and file. Tracks OCR status, stores extracted OCR text, and carries an `is_income` flag indicating the direction of the operation.
- **Transaction**: Monetary movement linking sender, receiver, source/destination accounts, and the receipt.
- **RecurringIncome**: Scheduled income entry that triggers automatic transaction registration on a defined period.

## Database schema

![Database schema](./assets/db-schema.png)

---

## Upload Receipt Workflow

1. The frontend uses the email configured in its `.env` to identify the active user.
2. User uploads a receipt image.
3. The CNN classifier identifies the originating bank.
4. The matching bank-specific OCR processor (Brubank, Lemon, or the LLM-based Default) extracts structured data.
5. The system persists the receipt and updates the affected account balances.

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

- receipt_router.py — POST /receipts/upload-receipt
  FastAPI router that receives the incoming HTTP request with the file, user_id, and is_income flag. Resolves service dependencies via injection and delegates the entire use case to ReceiptService.

- ReceiptService (receipt_service.py)
  Central orchestrator of the upload use case. Calls each infrastructure component in sequence — image conversion, bank classification, OCR extraction — then coordinates receipt persistence and delegates transaction creation and balance updates to the domain services. Guarantees that File and Receipt are persisted even if extraction fails.

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

---

## Considerations for Future Development

- Implementation of a real authentication and authorization flow (OAuth with Google).
- Support for receipts from additional financial institutions beyond Brubank and Lemon.
- Voice transaction input for logging transactions without receipts.
- Expense Reservations: setting money aside for upcoming expenses with future-spend visibility.
- Migration to dedicated blob storage (e.g., S3) for files.
- Timezone-aware date handling in the backend.
- Viewing and editing recurring income entries.
