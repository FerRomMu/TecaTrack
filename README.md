# TecaTrack Documentation

<p align="center">
  <img src="assets/app-icon.png" alt="TecaTrack Logo" width="350"/>
</p>

**Introduction**  
TecaTrack is an application designed for managing receipts and financial transactions using OCR (Optical Character Recognition) technology. Currently, the project is in a Proof of Concept (PoC) phase, specifically focused on processing receipts from Brunbank. The main objective is to extract structured information from receipt images and automatically map it to user accounts.

---

## MVP

### Minimum Features:

- **Receipt Upload**: Ability for the user to upload images of their receipts.
- **OCR Processing**: Extraction of key data from the receipt image.
- **Receipt Mapping**: Automatic association of the extracted data and updating of funds.
- **Balance Dashboard**: Visualization of the total balance (the sum of all user accounts) alongside the individual balance of each account, handled directly from the frontend with data obtained from the backend.

These features constitute the core of the system by demonstrating the feasibility of data extraction via OCR and the automatic synchronization of balances without the need for manual entry by the user.

---

## Premises/Requirements

Assumptions considered during the development of this PoC:

- **No Authentication (Login)**: There is no login flow in this PoC. Users are added manually to the database. The frontend determines which user to query using an environment variable (`.env`) that contains the active user's email.
- **OCR Scope**: The system is exclusively designed to process receipts from Brunbank.
- **User Identity**: Users are required to have a valid Argentine CUIL number.

---

## Architecture

The system follows a traditional client-server architecture, clearly separating responsibilities between the frontend and the backend.

- **Decoupled Frontend**: Single Page Application (SPA) developed with React and Vite.
- **Backend as REST API**: Developed in Python using the FastAPI framework.
- **Data Persistence**: PostgreSQL relational database.

## Repositories:

- [Frontend](https://github.com/FerRomMu/TecaTrack-Frontend)
- [Backend](https://github.com/FerRomMu/TecaTrack-Backend)

## Architecture Diagram

_(Empty for now)_

---

## Technologies Used

| Component      | Technology              | Reason                                               |
| -------------- | ----------------------- | ---------------------------------------------------- |
| Frontend       | React, Vite, TypeScript | Dynamic interface development and strict typing      |
| Frontend UI    | Ant Design (AntD)       | Avoid coding CSS and speed up the creation of styles |
| Backend        | Python, FastAPI         | Robust business logic and rapid API creation         |
| Database       | PostgreSQL, Alembic     | Relational data persistence and secure migrations    |
| Infrastructure | PaddleOCR               | Optical Character Recognition (OCR) to extract data from receipts |

---

## Domain Model

### Main Entities

- **User**: Represents the person in the system, uniquely identified by their CUIL.
- **Account**: Represents the bank account linked to the user, which maintains a money balance.
- **File**: Persisted binary representation of the receipt image uploaded by the user.

## Domain Diagram

_(Empty for now)_

---

## Main System Workflow

Usage flow in the current PoC:

1. The frontend system uses the email configured in its `.env` file to query the user (manually added in the DB).
2. The user uploads an image of a Brunbank receipt.
3. The system receives the image, persists it, and processes it through the OCR service.
4. The OCR extracts the data, maps it to a "Receipt", and automatically updates the money in the accounts.
5. The user views the updated balances (total and individual) on the dashboard.

---

## Project Structure

The project is divided into multiple repositories to maintain a clear separation of concerns:

- **TecaTrack (Main)**: Central repository used as a wiki, general project documentation, and technical reports (spikes).
- **TecaTrack-Frontend**: Contains all the user interface logic, views, components, and API integration.
- **TecaTrack-Backend**: Contains the REST API, business logic, OCR integration, and interaction with the PostgreSQL database.

---

## Important Technical Decisions

**Layered Architecture (Backend)**

The backend has been structured using a strict layered model (`routers`, `services`, `repositories`, `schemas`, `models`).  
**Reason**: To maintain a high separation of concerns, facilitate code maintainability, and allow incremental and atomic development.

**Use of Ant Design (Frontend)**

Ant Design (AntD) was chosen as the UI component library.  
**Reason**: The only reason for using Ant Design is to avoid writing CSS code manually, allowing the development of styles to be done much faster.

**Image Persistence (Backend)**

Images are persisted using the `BYTEA` data type in PostgreSQL.  
**Reason**: This approach was chosen for its simplicity following the KISS (Keep It Simple, Stupid) principle. While we may migrate to storing a reference to a dedicated blob storage in future versions if the system scales, persisting directly in the database is the simplest option for the current PoC.

---

## Considerations for Future Development

- Implementation of a real authentication and authorization flow (Login/Registration).
- Support for receipts from other banks or financial institutions.
- Voice transaction input: Allow users to upload or record an audio message with the transaction information instead of a receipt image.
- Expense Reservations: Ability to set money aside for upcoming expenses (configured as one-time entries, even for fixed or monthly installments) so they do not artificially inflate the available total balance.
