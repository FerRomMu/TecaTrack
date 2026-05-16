## Sprint 1 — Iteration Documentation

### Executive Summary

#### What was added in this iteration

- **Manual transaction system:** dedicated Upload section with a persistent NavBar, a form for manual transaction entry (type, origin/destination accounts, amount, date), and a POST endpoint on the backend.
- **Transaction history:** the dashboard now shows recent transactions with a "View more" button that opens a paginated modal. A GET /transactions endpoint was implemented with date-based cursor pagination.
- **Transaction filters:** combinable UI filters inside the transaction modal for date range, amount range, operation type (sent/received/all), and bank entity.
- **Edit transaction:** UI modal pre-filled with current values and a PATCH /transactions/:id endpoint that reverses the original balance impact before applying the new values.
- **Bank account creation:** POST /accounts endpoint with CBU/CVU validation and a frontend button on the dashboard that opens a modal form (bank name + CBU + balance amount).
- **User profile tab:** a new NavBar section at /user displaying the active user's name, email, and tax ID (CUIL), plus a user switcher for testing sender/receiver flows in development mode.
- **Temporary user listing endpoint:** a dev-restricted GET /user/all that returns all registered users for the frontend user switcher, guarded by NODE_ENV.
- **Receipt classifier:** backend component that inspects OCR text to identify the receipt origin (Brubank, Lemon, or Unknown) and routes processing to the appropriate processor.
- **Lemon receipt processor:** a dedicated processor with custom OCR bounding-box rules and field mapping specific to Lemon transfer receipts.
- **Default receipt processor:** a generic fallback processor that uses keyword-based extraction (amounts, dates, CBU patterns) for receipts that no specific processor handles.
- **Income receipt handling:** logic within the receipt pipeline that compares sender and receiver CBUs against the user's registered accounts to classify the operation as income, expense, or internal transfer.
- **Recurring income (frontend):** a form inside the Upload section to schedule periodic income entries, including account selector, amount, frequency (weekly/biweekly/monthly), and start date.
- **Recurring income (backend):** new database schema, POST /accounts/recurring-income endpoint, and a worker script to process recurring income transactions and update account balances automatically.
- **API client refactor:** migration from native fetch to Axios for centralized base URL, headers, interceptor support, and automatic serialization.

#### Decisions made

- **Income detection via user flag instead of OCR:** after analyzing receipts, most do not include enough sender information to determine transfer direction. The system now relies on a user-provided flag to mark a receipt as income.
- **Axios selected:** chosen over alternatives due to team familiarity and strong interceptor support.
- **NavBar-based navigation:** replaced single dashboard layout with persistent navigation.
- **Development-only user endpoint:** GET /user/all restricted via NODE_ENV.
- **Added classification system for receipts:** The system classifies receipts acording which bank they are from (Brubank, Lemon, or Unknown) to know how to extract data from them.
- **Added default processor for receipts with LLMs:** We added a free Gemini API key to process receipts that are not from Brubank or Lemon. This is a solution for unknown banks.  

---

## User Stories


### Create manual transaction

**Actor:** User  
**Functionality:** Submit a transaction and update balances automatically  
**Value:** Keeps financial records complete  

**Acceptance criteria:**
- Form includes type, accounts, amount, and date
- Success message is shown
- Backend receives data
- Dashboard updates balances

---

### Bank account creation

**Actor:** User  
**Functionality:** Add a bank account from dashboard  
**Value:** Enables multi-account tracking  

**Acceptance criteria:**
- "Add account" button visible
- Modal form opens
- CBU validates 22 digits
- Dashboard refreshes after creation

---

### View transactions

**Actor:** User  
**Functionality:** View recent and full transaction history  
**Value:** Quick access and full visibility  

**Acceptance criteria:**
- Recent transactions visible
- "View more" adds next page
- Scroll supported
- Pagination implemented

---

### Transaction filters

**Actor:** User  
**Functionality:** Filter transaction list  
**Value:** Quickly locate specific records  

**Acceptance criteria:**
- Filter panel present
- Bank selector populated dynamically
- Type toggles available
- Amount and date filters supported
- Filters are combinable

---

#### User tab

**Actor:** User  
**Functionality:** View and switch users  
**Value:** Supports development testing  

**Acceptance criteria:**
- Profile section available
- Displays name, email, and tax ID
- User switcher present
- Loading indicator shown

---

#### Edit transaction

**Actor:** User  
**Functionality:** Modify transaction data  
**Value:** Correct errors without re-entry  

**Acceptance criteria:**
- Edit button available
- Modal pre-filled
- Balances recalculate automatically

---

### Recurring income

**Actor:** User  
**Functionality:** Schedule recurring income  
**Value:** Automates regular entries  

**Acceptance criteria:**
- Form includes amount, account, description
- Frequency selector available
- Start date defined

---

### Upload receipt income 

**Actor:** User  
**Functionality:** Upload receipt and classify as income
**Value:** Saves time when receiving income instead of manual uploads

**Acceptance criteria:**
- User can upload a receipt
- Income balance updated correctly

---

## Technical Tasks

### Manual transaction layout

**Actor:** User  
**Functionality:** Access a dedicated Upload section for manual or receipt-based operations.  
**Value:** Centralized data entry without cluttering the dashboard.

**Acceptance criteria:**
- Persistent NavBar switches between /dashboard and /upload
- Upload page centralizes data entry
- Receipt upload component moved from Dashboard
- Manual form includes type, accounts, amount, and date
- Active section is visually highlighted

---

### Manual transaction endpoint

**Actor:** System  
**Functionality:** Create transactions via API  
**Value:** Enables persistence and balance updates  

**Acceptance criteria:**
- Balances update automatically
- UUID returned

---

### Bank account endpoint

**Actor:** System  
**Functionality:** Create account  
**Value:** Enables multi-account support  

**Acceptance criteria:**
- Validates data
- Returns created account

---

### Retrieve transactions

**Actor:** System  
**Functionality:** List transactions with filters  
**Value:** Supports frontend queries  

**Acceptance criteria:**
- Pagination supported
- Filters implemented

---

### User listing endpoint

**Actor:** System  
**Functionality:** List users in dev mode  
**Value:** Enables testing  

**Acceptance criteria:**
- Returns basic user data

---

### Update transaction endpoint

**Actor:** System  
**Functionality:** Modify transactions  
**Value:** Maintains consistency  

**Acceptance criteria:**
- Reverses and reapplies balances

---

### Receipt classifier

**Actor:** System  
**Functionality:** Detect receipt origin  
**Value:** Improves OCR processing  

---

### Lemon processor

**Actor:** System  
**Functionality:** Handle Lemon receipts  
**Value:** Higher extraction accuracy  

---

### Default processor

**Actor:** System  
**Functionality:** Handle unknown receipts  
**Value:** Ensures fallback processing  

---

### Receipt income classification

**Actor:** System  
**Functionality:** Detect transaction type  
**Value:** Correct balance handling  

---

### Recurring income backend

**Actor:** System  
**Functionality:** Automate scheduled transactions  
**Value:** Reduces manual entry  

---

### API client refactor

**Actor:** Developer  
**Functionality:** Replace fetch with Axios  
**Value:** Improve maintainability  
 
