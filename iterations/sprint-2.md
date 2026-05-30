# Sprint 2 — Iteration Documentation

## Executive Summary

### What was added in this iteration

- Secure sign-in: users log in with their Google account, and the whole app is protected so each person only reaches their own financial data.
- Personal categories: users create their own categories and assign them to transactions and recurring incomes, then filter by category.
- Reviewed receipt capture: after uploading a receipt, the system shows what it read so the user can check and correct it before saving — fewer errors from automatic reading.
- Correct local dates: transactions are recorded in the user's own time zone, and future dates are not allowed.
- Recurring income management: users can view, edit, and filter their scheduled incomes in a dedicated section.
- Dedicated transactions view: transactions now have their own section, separate from the dashboard.

### Decisions made

- Delegated login instead of our own: Google sign-in avoids another password and storing credentials, improving security.
- Review-before-save for receipts: automatic reading can misinterpret a receipt, so a confirmation step keeps data accurate.
- Identity tied to the logged-in user: each action is bound to the authenticated user, so people only see and change their own data.
- Time-zone awareness: dates are stored in each user's local time so records stay accurate regardless of the server.
- Per-user categories: categories belong to each user and behave the same across transactions and recurring incomes.

---

## User Stories
