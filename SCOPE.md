# CSV Import Anomaly Scope & Policies

This document outlines the validation rules, detected anomalies, and error-handling policies for the Splitwise Clone CSV Import feature.

## 1. CSV Data Format Specifications
The importer expects a CSV file containing the following columns (matched case-insensitively, ignoring non-alphanumeric characters):

| Column | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| **`Date`** | YYYY-MM-DD | Yes | The calendar date of the expense. Cannot be in the future. |
| **`Description`** | String | No | Description of the expense. |
| **`Group`** | String | Yes | Name or UUID of the group. Must exist. |
| **`Amount`** | Decimal | Yes | Total cost of the expense. Must be positive (> 0). |
| **`Paid By`** | String (Email) | Yes | Email of the payer. Must exist in group. |
| **`Split Type`** | String | Yes | One of: `EQUAL`, `UNEQUAL`, `PERCENTAGE`, `SHARE`. |
| **`Splits`** | String | Yes | Split details (e.g., `alice@example.com:50;bob@example.com:50`). |

---

## 2. Supported Anomalies and Actions
The system checks every imported row and logs anomalies in the `import_anomalies` table. Below are the specific anomaly codes, descriptions, and recovery actions:

### 1. `INVALID_DATE`
* **Condition:** Date column is empty, malformed, or represents a future calendar date.
* **Action:** **Skip Row** (to prevent database integrity/logic issues).

### 2. `MISSING_DESCRIPTION`
* **Condition:** Description is blank or empty.
* **Action:** **Pending Approval** (Assigned default description `Imported Expense - Row [N]`, sent to user to approve or reject).

### 3. `INVALID_AMOUNT`
* **Condition:** Amount column is not a number, empty, or <= 0.
* **Action:** **Skip Row** (financial transactions must have positive value).

### 4. `GROUP_NOT_FOUND`
* **Condition:** Group name or UUID specified does not match any existing group.
* **Action:** **Skip Row** (expenses must belong to a valid active group).

### 5. `PAYER_NOT_FOUND`
* **Condition:** Payer email is not registered on the platform, or is registered but is not a member of the target group.
* **Action:** **Skip Row** (only group members can pay for expenses).

### 6. `INVALID_SPLITS`
* **Condition:** Splits data is empty, or split values are mathematically incorrect:
  * For `UNEQUAL` split type: Sum of split amounts != total expense amount.
  * For `PERCENTAGE` split type: Sum of split percentages != 100%.
  * For `SHARE` split type: Sum of split shares <= 0.
* **Action:** **Skip Row** (prevents debt calculation imbalances).

### 7. `PARTICIPANT_NOT_FOUND`
* **Condition:** One or more split participants are not registered users, or are not members of the target group.
* **Action:** **Skip Row** (prevents splitting expenses with external or non-member users).

### 8. `DUPLICATE_EXPENSE`
* **Condition:** An expense with the same description, amount, group, payer, and date already exists in the database.
* **Action:** **Pending Approval** (prevents duplicate charging without explicit user approval).

### 9. `SETTLEMENT_ROW`
* **Condition:** CSV row represents a settlement/payment instead of a normal expense (e.g., description matches `payment`, `settlement`, `refund`, `transfer`, `repayment`, `payback`).
* **Action:** **Pending Approval** (row is not directly created as Expense; ImportApproval is created and after approval it becomes a Settlement).

---

## 3. Database Schema

Based on the actual Prisma schema, the following models are used:

### 1. `User`
* **Purpose:** Represents a registered user on the platform.
* **Important Fields:** `id`, `email`, `passwordHash`, `name`, `createdAt`.
* **Relationships:** Can create groups, be a group member, pay expenses, participate in splits, and send/receive settlements.
* **Why it exists:** Core authentication and identity tracking.

### 2. `Group`
* **Purpose:** Represents a shared group where expenses are split.
* **Important Fields:** `id`, `name`, `creatorId`, `createdAt`.
* **Relationships:** Belongs to a creator (`User`), contains `members` (`GroupMember`), `expenses`, and `settlements`.
* **Why it exists:** Isolates expenses and debts among a specific set of users.

### 3. `GroupMember`
* **Purpose:** Tracks a user's membership in a group over time.
* **Important Fields:** `isActive`, `joinedAt`, `leftAt`.
* **Relationships:** Links a `User` to a `Group`.
* **Why it exists:** Implements soft-deletion so historical expenses remain mathematically correct even if a member leaves.

### 4. `Expense`
* **Purpose:** Represents a single group expense.
* **Important Fields:** `amount` (converted), `originalAmount`, `originalCurrency`, `convertedAmount`, `description`, `createdAt`.
* **Relationships:** Linked to a `Group` and a `User` (payer). Has multiple `splits`.
* **Why it exists:** The core financial record driving debts in the group.

### 5. `ExpenseSplit`
* **Purpose:** Details how an expense is divided among participants.
* **Important Fields:** `amount`, `splitValue`, `splitType` (`EQUAL`, `UNEQUAL`, `PERCENTAGE`, `SHARE`).
* **Relationships:** Linked to an `Expense` and a `User` (participant).
* **Why it exists:** Stores the exact debt portion each user owes for an expense.

### 6. `Settlement`
* **Purpose:** Records a peer-to-peer repayment within a group.
* **Important Fields:** `amount`, `createdAt`.
* **Relationships:** Linked to a `Group`, a `sender` (payer), and a `receiver`.
* **Why it exists:** Offsets group debts without creating a shared expense.

### 7. `ImportAnomaly`
* **Purpose:** Logs validation errors or soft anomalies during CSV imports.
* **Important Fields:** `rowNumber`, `anomalyType`, `description`, `actionTaken`.
* **Relationships:** Belongs to an `Import` and can have `approvals`.
* **Why it exists:** Provides an audit trail for failed or flagged import rows.

### 8. `ImportApproval`
* **Purpose:** Holds row data for soft anomalies awaiting user action.
* **Important Fields:** `action`, `status` (`PENDING`, `APPROVED`, `REJECTED`), `rowData`, `approvedBy`.
* **Relationships:** Linked to an `ImportAnomaly`.
* **Why it exists:** Powers the interactive approval workflow for settlements, duplicates, and missing descriptions.
