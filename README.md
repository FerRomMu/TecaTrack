# TecaTrack Documentation

<p align="center">
  <img src="assets/app-icon.png" alt="TecaTrack Logo" width="350"/>
</p>

**Introduction**  
TecaTrack is an application for managing receipts and financial transactions using OCR (Optical Character Recognition) technology. It extracts structured information from receipt images and maps it to user accounts, complementing manual and recurring entries. The project has grown beyond its initial Proof of Concept: it now supports authenticated multi-user access, multiple banks, category management, and recurring income.

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

---

## Important Technical Decisions

**Layered Architecture (Backend)**
Structured using `routers`, `services`, `repositories`, `schemas`, and `models`, with an `infrastructure` layer for OCR, the classifier, and the LLM client, to maintain separation of concerns and allow atomic development.

**Image Persistence (Backend)**
Persisted using `BYTEA` in PostgreSQL following the KISS principle for the current scope, with potential to migrate to blob storage later.

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
- Expense Reservations: setting money aside for upcoming expenses.
- Additional bank-specific OCR processors beyond Brubank and Lemon.
- Migration to dedicated blob storage (e.g., S3) for files.
- Multi-language support (currently Spanish/Argentina only).
