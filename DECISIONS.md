# Engineering Decisions

This document summarizes the architectural and implementation decisions made during the development of the Splitwise Clone and the CSV Import module.

## 1. Database Schema Extensions & Migration
* **Decision:** Appended `Import` and `ImportAnomaly` models to the existing database schema and executed dev migrations.
* **Options considered:**
  * Store import logs only in memory
  * Store import logs in database
  * Create separate microservice
* **Why this option was chosen:** Keeping import metadata and anomaly logs in the database allows for audit trails, historical report viewing, and robust E2E validation reporting.

## 2. Decoupled Split Calculation Logic
* **Decision:** Extracted the split-value arithmetic (including remainder adjustments to eliminate precision errors) into a exported helper `calculateSplits` in `expenseController.js`.
* **Options considered:**
  * Duplicate calculation logic
  * Put logic in controller only
  * Extract reusable helper
* **Why this option was chosen:** Allows the manual expense creator and the CSV batch import processor to compute debts in an identical way, ensuring consistency in precision rounding.

## 3. Custom CSV Parser vs. External Library
* **Decision:** Implemented a custom CSV parser with double-quote support and auto-trimming in `csvParser.js`.
* **Options considered:**
  * Use csv npm library
  * Use manual parser
  * Pre-process CSV manually
* **Why this option was chosen:** Reduces runtime dependencies and avoids platform-specific compilation issues on developer machines (e.g. windows native bindings) while maintaining high speed and reliability.

## 4. Dashboard CSV Import Modal Integration
* **Decision:** Integrated the CSV Import flow as a rich animated overlay modal on the user Dashboard.
* **Options considered:**
  * Separate import page
  * Modal based import
  * Background import service
* **Why this option was chosen:** Prevents layout fragmentation and provides immediate feedback. Users can import files, review log anomalies instantly, click "Done & Refresh" to trigger React state re-fetching, and see updated balances and groups on the fly.

## 5. Migration to Local PostgreSQL
* **Decision:** Migrated `DATABASE_URL` in the dev environment to a local PostgreSQL instance.
* **Options considered:**
  * Continue Neon cloud database
  * Use local PostgreSQL
  * Use another database provider
* **Why this option was chosen:** Eliminated Neon cloud pooler timeouts, ensuring stable database transactions during heavy batch inserts.

## 6. Vercel SPA Routing Configuration
* **Decision:** Added a `vercel.json` file with wildcard rewrite rules pointing to `index.html`.
* **Options considered:**
  * Change frontend routing
  * Add server rewrites
  * Use vercel.json configuration
* **Why this option was chosen:** Fixes the standard React SPA issue on Vercel where browser page refreshes on sub-routes (`/dashboard`, `/group/:id`) throw Vercel 404 NOT FOUND errors.

## 7. CSV Settlement Import Anomaly Detection
* **Decision:** CSV rows representing settlements/payments (detected by keywords in the description) are flagged as `SETTLEMENT_ROW` anomalies and routed through the existing approval workflow.
* **Options considered:**
  * Import settlements as expenses
  * Ignore settlement rows
  * Detect and route through approval workflow
* **Why this option was chosen:** Prevents importing peer-to-peer debt settlements as group expenses, preserving ledger integrity, while allowing users to approve and record them correctly.
