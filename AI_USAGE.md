# AI Tools Used

During the development of this Splitwise Clone, the following AI tools were utilized:
- **ChatGPT** (OpenAI)
- **Antigravity** (Google DeepMind Advanced Agentic Coding Assistant)

AI assistance was instrumental across multiple phases of the project:
- **Code Review**: AI helped perform comprehensive requirement audits against the assignment guidelines, identifying missing edge cases.
- **Debugging**: AI assisted in tracing and resolving issues such as Prisma schema mismatches and API route failures.
- **Documentation**: AI was used to draft and format structural documents like `SCOPE.md`, `DECISIONS.md`, and `README.md`.
- **Architecture Discussions**: AI helped brainstorm and design decoupled logic for split calculations, multi-currency conversion, and the CSV anomaly approval workflow.

---

# Key Prompts Used

Below are examples of key prompts used during development across different categories:

1. **Architecture planning prompt**
   - *"Design a relational database schema for a Splitwise clone that supports soft deletion for group members, ensuring historical expenses remain mathematically correct even if a member leaves the group."*

2. **Code review/audit prompt**
   - *"You are a senior software engineer reviewing my completed project. Perform a complete requirement audit of the Splitwise clone codebase against every requirement from the assignment. Mark status and explain where it exists."*

3. **CSV importer/anomaly handling prompt**
   - *"Implement a custom CSV parser and anomaly detector for expense imports. It must validate dates, amounts, descriptions, and verify that the group, payer, and all participants actually exist before importing."*

4. **Database schema review prompt**
   - *"Review my Prisma schema. Verify that the 'ImportApproval' and 'ImportAnomaly' models correctly link together and can support JSON payloads for row data."*

5. **Bug fixing prompt**
   - *"Fix the 'Cannot read properties of undefined (reading 'create')' error occurring during CSV expense imports in the Splitwise clone application. Check the Prisma client mapping and transactions."*

---

# AI Mistakes & Corrections

## Case 1: Incorrect Member Deletion Approach

**AI Output:**
AI suggested deleting group members permanently from the database when they leave a group.

**How detected:**
Reviewed assignment requirement and noticed historical expenses would break if the user record was completely removed from the group.

**Fix:**
Implemented soft deletion using:
- `isActive`
- `leftAt`

---

## Case 2: CSV Negative Amount Handling

**AI Output:**
AI sanitization logic converted negative amounts into positive values.

**How detected:**
Code review of CSV parser showed `"-"` sign removal in the regex `/^[^0-9.]/g`.

**Fix:**
Preserved negative values and added anomaly detection to explicitly flag them as invalid amounts.

---

## Case 3: Settlement Imported As Expense

**AI Output:**
AI treated payment/settlement rows as normal expenses.

**How detected:**
Compared CSV business rules with expense calculation flow and realized settlements were throwing off group debt balances.

**Fix:**
Added `SETTLEMENT_ROW` anomaly detection and routed it through the approval workflow to be converted into a `Settlement` record rather than an `Expense`.

---

# Final Notes

All AI-generated suggestions were reviewed manually. No AI output was accepted without checking against:
- assignment requirements
- database design
- application logic
