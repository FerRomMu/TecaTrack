# Changelog

All notable changes to this repository will be documented in this file.

This project follows **Semantic Versioning (SemVer)**.

---

## [0.4.0] - 2026-06-12 - Sprint 3

### Added

- Documentation for the OCR microservice (`apps/ocr/`) and the `OCR_SERVICE_URL` environment variable.
- Documentation for the asynchronous receipt pipeline (`202 Accepted`, background processing, `PENDING` state) and the receipt list with per-receipt status and inline confirmation.
- Documentation for movement statistics (time series, rankings by category and by bank) and the `@ant-design/charts` dependency.
- Sprint 3 iteration report (`iterations/sprint-3.md`).

### Changed

- Renamed the Recurring Income domain to Recurring Movements (income + expenses), documenting `movement_type`, `is_active`, `expires_at`, and deactivation.
- README updated: features, environment variables, technologies table, domain vocabulary, upload workflow, and component responsibilities now reflect the OCR microservice split and the async receipt flow.
- Important Technical Decisions extended with OCR-as-a-microservice, asynchronous receipt processing, recurring-movement generalization, and chart-library selection.
- Database schema diagram and architecture diagram regenerated for the OCR microservice split and the `recurring_movements` rename.

---

## [0.3.0] - 2026-05-29 - Sprint 2

### Added

- Documentation for Google OAuth 2.0 + JWT authentication across the ecosystem.
- Documentation for the Categories domain (CRUD, assignment, and filtering on transactions and recurring incomes).
- Documentation for the two-phase receipt upload → confirm flow with OCR readout review.
- Documentation for multi-bank OCR (Brubank, Lemon, and the Gemini LLM fallback) and the CNN receipt classifier.
- Documentation for timezone-aware transactions and future-date validation.
- New domain entities documented: Category and Recurring Income.
- Sprint 2 iteration report (`iterations/sprint-2.md`).

### Changed

- README rewritten to reflect the post-PoC feature set (authentication, categories, recurring income, transaction management).
- Premises updated: authentication is now required (Google OAuth) and OCR supports multiple banks.
- Technologies table extended with the Gemini LLM, the CNN classifier (EfficientNet-B0), Google OAuth/JWT, React Router, and Axios.
- Main system workflow updated for OAuth login and the upload/confirm receipt flow.
- Architecture section updated to describe the backend infrastructure layer (OCR, classifier, LLM) and background workers.
- Data model (`doc/poc-datamodel.dbml`) updated with the categories table, receipt confirmation fields, and timezone-aware transaction columns.

---

## [0.2.0] - 2026-05-16 - Sprint 1

### Added

- Iteration summary documented in `iterations/sprint-1.md` and `iterations/poc.md`.
- Diagram architecture based on a specific user story.

---

## [0.1.0] - 2026-04-24 - DEMO

### Added

- Initial documentation repository structure.
- TecaTrack ecosystem overview and features documentation.
- Architecture description and architecture diagram.
- Domain concepts and main entities description.
- Technical decisions section.
- Future development considerations.
- Database schema documentation.
- Main system workflow description.
- Versioning strategy documentation.

### Notes

- This is the first documentation release for the TecaTrack ecosystem PoC.
