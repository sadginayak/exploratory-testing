# Session-Based Test Charter — Monefy mobile application

## Header
- **Application:** Monefy Mobile application
- **Tester:** Sadgi Nayak
- **Date:** 10.09.2026
- **Time-box:** 120 min
- **Device:** OnePlus 7T(Android)
- **App:** Monefy v2.1.5
- **Testing Approach:** Black-box exploratory testing

---
###  Charter 1: Multi-Currency Carry-Over Validation
* **Category:** Data Integrity
* **Time-Boxing:** 20 Minutes
* **Mission:** Explore Monefy with active "Carry Over" logic enabled to discover how currency configuration changes impact historical balance.

**Preconditions:**
- Fresh install, zero transactions
- Currency: INR (₹)
- Carry-over: ENABLED

**Steps:**

1. Add ₹1,000 to previous month
3. Navigate to current month; verify carry-over shows ₹1,000
4. Add ₹30,000 to current month; verify total shows ₹31,000
5. Go to Settings, change currency to EUR (€)
6. Return to dashboard, check both month balances

**Expected vs Actual:**
| Step | Expected | Actual | Status |
|------|----------|--------|--------|
| 4 | Total: ₹31,000 | ₹31,000 | ✅ |
| 6 | Previous month: ~€12.50 (converted) | €1,000 (no conversion) | ❌ FAIL |
| 6 | Current month: ~€387.50 | €31,000 | ❌ FAIL |

####  Observations:
* Previous month showed `1,000 EUR` and current month showed `31,000 EUR`. No mathematical conversion, amounts unchanged, only currency symbol swapped.


####  Identified Risks - CRITICAL:
* Silent financial data corruption. User believes they have €31,000 when actual converted value is less.
* The app lacks warning modals informing users that global currency swaps are cosmetic text switches rather than financial data conversions, breaking tracking accuracy.
----
## Charter 2.1: Budget Mode - Income Display

* **Category:** UX / Information Architecture
* **Time-Box:** 20 minutes
* **Mission:** Explore Budget mode with new income as categories(Deposit, Salary and Savings) to discover funds calculation and distribution across expenses.

**Preconditions:**
- Currency: EUR (€)
- Budget mode: disabled

**Steps:**
1. Enable Budget mode, set limit 30,000 EUR
2. Dashboard shows 30,000 EUR
3. Add income #1: 20,000 EUR (Savings category)
4. Add income #2: 20,000 EUR (Salary category)
5. Add income #3: 20,000 EUR (Deposit category)
6. Check main dashboard balance

**Expected vs Actual:**
| Step | Expected | Actual | Status |
|------|----------|--------|--------|
| 2 | Shows budget limit: 30,000 EUR | 30,000 EUR | ✅ |
| 6 | Shows total: 60,000 EUR (3x income) | 30,000 EUR (unchanged) | ❌ FAIL |
| 6 | Clicking Balance button reveals details | Shows all 3 sources + limit | ✅ |

**Observation:**
1. Without any income, on setting budget mode with spending limit,on main Dashboard and in Balance 30000.00 euros are displayed. Which confused me thinking it is available balance, however on clicking balance there is no income source added.

2. After adding 60,000 EUR income, dashboard still shows 30,000 EUR. Main balance unaffected by income additions. Clicking Balance button displays all income sources and budget limit.

**Risk - MEDIUM:**
Budget limit and available balance conflated. User confused whether 30,000 EUR is available balance or spending cap. Reduces financial clarity.

**Question:**
Is Budget mode designed to show only limit (ignoring income), or should it display total available funds?

---
## Charter 2.2: Budget Limit Scope - Per-Month vs Global

* **Category:** Business Logic
* **Time-Box:** 15 minutes
* **Mission:** Explore Budget mode with changing budget limit to discover if its month specific or global.

**Preconditions:**
- Current month: September 2026
- Previous month: August 2026
- Enable Budget mode, set limit for current month: 30,000 EUR

**Steps:**
1. Add income: 20,000 EUR
3. Scroll to previous month
4. Enable Budget mode for previous month, set limit: 20,000 EUR
5. Return to current month, verify limit still 30,000 EUR
6. Try to set budget for future month

**Expected vs Actual:**
| Step | Expected | Actual | Status |
|------|----------|--------|--------|
| 1 | Current month limit: 30,000 EUR | 30,000 EUR | ✅ |
| 4 | Previous month limit: 20,000 EUR only | 20,000 EUR | ✅ |
| 5 | Current month still: 30,000 EUR | 20,000 EUR (changed) | ❌ FAIL |
| 6 | Future month option available | No option to set future month budget | ❌ FAIL |

**Observation:**
Setting budget limit for previous month (20,000 EUR) changed limit for ALL months, including current month (was 30,000 EUR, now 20,000 EUR). No option to set budget for future months.

**Risk - MEDIUM:**
Budget limit is global, not per-month. User cannot isolate spending limits by month. Unrealistic for varying monthly expenses (holidays, seasonal spending).

**Root Cause Hypothesis:**
Budget limit stored as global setting, not indexed by month.

---
## Charter 2.3: Budget Exceeded - No Warning

* **Category:** Data Integrity / Financial Logic
* **Time-Box:** 20 minutes
* **Mission:** Explore Budget mode with adding expense more than income to discover if it flags unavailable balance or Budget exceeded

**Preconditions:**
- Currency: EUR (€)
- Budget mode: enabled

**Steps:**
1. Set budget limit: 40,000 EUR
2. Add income in current month (Sep 2026): 20,000 EUR (Deposit category)
3. Add expense in current month (Sep 2026): 35,000 EUR (Car category)
4. Add expense in current month (Sep 2026): 5,000 EUR (Clothes category)
5. Add expense in current month after exhausting Budget spending limit (Sep 2026): 5,000 EUR (Food category)


**Expected vs Actual:**
| Step | Expected | Actual | Status |
|------|----------|--------|--------|
| 2 | Available balance: 20,000 EUR on Dashboard | Shows 40,000 EUR | ❌ FAIL |
| 3 | Warning: "Exceeds available balance" OR error | Balance shows 5,000 EUR | ❌ FAIL |
| 4 | Warning: "Insufficient funds" OR error | Balance shows 0 EUR | ❌ FAIL |
| 5 | Warning: "Budget limit exceeded for the month" | Balance shows -5,000 EUR | ❌ FAIL |


**Observation:**
With 20,000 EUR income, user added 45,000 EUR expenses. Dashboard calculated balance against budget limit (40,000 EUR), not actual income. Final balance: -5,000 EUR (negative). No warning before or after adding expenses.

**Risk - CRITICAL:**
App displays negative balance without warning. User believes they have funds when actually in deficit. Balance calculation: (Budget Limit - Expenses) instead of (Income - Expenses). Creates financial blind spot. If linked to real bank account, overdraft fees possible.

**Root Cause Hypothesis:**
Balance logic: available_balance = budget_limit - total_expenses (ignores income). Should be: available_balance = income - total_expenses.

**Automation Candidate:** YES - Regression test to prevent negative balances and require warnings before overdraft.

---

## Charter 3: Navigation Context Mismatch - Add Expense/Income

* **Category:** UX / Navigation Bug
* **Time-Box:** 10 minutes
* **Mission:** Explore previous months with "Add Expense" / "Add Income" feature to discover changes reflect on edited month.

**Preconditions:**
- App open, current month visible
- Multiple months available
- Fresh/existing transactions

**Steps:**
1. Navigate to previous month.(Current Sep 2026)
2. Verify month label shows "August"
3. Click "Add Expense" button
4. Enter expense details (amount, category)
5. Verify in which month expense appears

**Expected vs Actual:**
| Step | Expected | Actual | Status |
|------|----------|--------|--------|
| 2 | Month label: "August" | August displayed | ✅ |
| 3-4 | Add Expense form opens | Form opens | ✅ |
| 5 | Expense added to August | Expense added to current month (Spetember) | ❌ FAIL |
| 5 | Date field auto-populated to August | Date must be manually changed to August | ❌ FAIL |

**Observation:**
User navigates to previous month . Month label and UI context show accordingly. Clicking "Add Expense" opens form but adds transaction to current month. User must manually select date field and change it to previous month each time.

**Risk - MEDIUM (User Impact: HIGH):**
UX illusion: UI context (month label) doesn't match button behavior (adds to current month). Causes accidental transactions in wrong month. Frustrating workflow.

**Root Cause Hypothesis:**
"Add Expense" button hardcoded to use current_month variable instead of selected_month variable. UI updates but button action doesn't respect navigation state.

**Automation Candidate:** YES - Test: Navigate to Month X, Add Expense, verify expense appears in Month X (not current month).

---
## Charter 4: Data Persistence Across App Restart

* **Category:** Data Integrity / Reliability
* **Time-Box:** 8 minutes
* **Mission:** Verify transactions persist after unexpected app termination

**Steps:**
1. Add 5 transactions in current month with distinct amounts and categories
2. Force-stop app: Settings > Apps > Monefy > Force Stop
3. Reopen app
4. Verify all 5 transactions visible with correct amounts

**Expected vs Actual:**
| Step | Expected | Actual | Pass? |
|------|----------|--------|-------|
| 3 | All 5 transactions visible | All 5 transactions present ✅ | ✅ PASS |
| 3 | Amounts unchanged | (income)1000.00, 0.99, 0.01, 1(expense clothes), 20000(expense car) ✅ | ✅ PASS |
| 3 | Balance correct | 19000 (income - expenses) ✅ | ✅ PASS |

**Observation:**
After force-close, all transactions persisted correctly. No data loss.

**Risk:** NONE (persistence works correctly)



## Appendix: Database Verification Attempt
* **Time-Box:** 15 minutes

**Objective:** Verify that currency conversion amounts are correctly stored in database 
(not just UI display change)

**Method:** 
1. Connected device via USB debugging
2. Used ADB to pull database file: `/data/data/com.monefy.app.lite/`
3. Attempted to open `.db` file in SQL viewer (DB Browser for SQLite)

**Findings:**
- Database file could not be extracted due to permission restrictions
- No visible tables in pulled database files

**Implication:**
- Could not definitively prove currency amounts stored incorrectly in DB
- Findings based on UI observation only


