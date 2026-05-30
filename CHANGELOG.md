# Changelog

All notable changes to this repository will be documented in this file.

This project follows **Semantic Versioning (SemVer)**.

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

### Notes

- The `0.2.0` documentation release was skipped; this entry brings the documentation hub from `0.1.0` directly to the `0.3.0` ecosystem state.

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
