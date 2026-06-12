# AI Usage & Collaboration

This document outlines how the AI developer assistant (Antigravity) was utilized to implement and test the Splitwise clone features.

## 1. Task Definition & Plan Generation
* The AI generated and updated structured plans (`BUILD_PLAN.md` and `AI_CONTEXT.md`) based on user specifications.
* The system boundaries, anomaly rules, and database schema mappings were established cooperatively prior to code changes.

## 2. Code Generation & Iterative Refactoring
* **Backend:** The AI generated the custom CSV parser (`csvParser.js`), validation engine (`anomalyDetector.js`), and import service (`importService.js`).
* **Frontend:** The AI updated the main Dashboard view (`Dashboard.jsx`) to embed the CSV file input dropzone, processing status banners, and reactive anomaly validation reports.
* **Refactoring:** The AI identified duplicate logic in split calculation and updated `expenseController.js` to export a unified helper `calculateSplits`, reducing code duplication.

## 3. Database Migration & Local Environment Configuration
* The AI generated schema modifications for the PostgreSQL database via Prisma client models.
* Assisted in troubleshooting connection pool issues by migrating database credentials to a local PostgreSQL container, stabilizing development.

## 4. End-to-End Testing & Verification
* The AI designed and executed a local NodeJS integration script (`backend/src/testImport.js`) to automatically verify user/group seeding, transaction logic, rounding correction, and each of the 8 anomaly edge-cases.
* Created a sample spreadsheet file (`expenses_export.csv`) to test file uploads inside the browser environment.
