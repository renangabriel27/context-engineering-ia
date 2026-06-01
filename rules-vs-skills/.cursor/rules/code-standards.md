# Coding Standards

## 1. Source Code Language
All source code must be written in English.

*   **Before:** `const precoTotal = 100;`
*   **After:** `const totalPrice = 100;`

---

## 2. Naming Conventions
Use `camelCase` for methods, functions, and variables; `PascalCase` for classes and interfaces; and `kebab-case` for files and directories.

*   **Before:** `function calculate_total() {}`, `class user_account {}`, `file_name.js`
*   **After:** `function calculateTotal() {}`, `class UserAccount {}`, `file-name.js`

---

## 3. Abbreviations and Name Length
Avoid abbreviations, but do not use names longer than 30 characters.

*   **Before (Abbreviation):** `const usrAddr = '...';`
*   **Before (Too Long):** `const userAccountShippingAddressStreetNameAndNumber = '...';`
*   **After:** `const userAddress = '...';`

---

## 4. Magic Numbers
Declare constants for magic numbers to improve readability.

*   **Before:** `if (status === 1) { ... }`
*   **After:**
    ```javascript
    const STATUS_ACTIVE = 1;
    if (status === STATUS_ACTIVE) { ... }
    ```

---

## 5. Method and Function Names
Methods and functions must perform a clear action, and their names must start with a verb.

*   **Before:** `function total() { ... }`
*   **After:** `function calculateTotal() { ... }`

---

## 6. Parameters Limit
Avoid passing more than 3 parameters; prefer using objects instead.

*   **Before:** `function createUser(name, email, age, address, phone) { ... }`
*   **After:** `function createUser({ name, email, age, address, phone }) { ... }`

---

## 7. Side Effects (Command Query Separation)
Avoid side effects. A method should either perform a mutation (command) or a query, but never both.

*   **Before (Query with side effect):**
    ```javascript
    function getBalance(account) {
      logAccess(account); // Side effect in a query
      return account.balance;
    }
    ```
*   **After:**
    ```javascript
    function getBalance(account) {
      return account.balance;
    }
    ```

---

## 8. If/Else Nesting and Early Returns
Never nest more than two `if/else` blocks. Use early returns instead.

*   **Before:**
    ```javascript
    if (user) {
      if (user.isActive) {
        if (user.hasPermission) {
          return accessGranted();
        }
      }
    }
    ```
*   **After:**
    ```javascript
    if (!user) return;
    if (!user.isActive) return;
    if (!user.hasPermission) return;
    return accessGranted();
    ```

---

## 9. Flag Parameters
Never use flag parameters to switch behavior. Extract them into specific functions.

*   **Before:**
    ```javascript
    function toggleUserStatus(user, isAdmin) {
      if (isAdmin) { ... } else { ... }
    }
    ```
*   **After:**
    ```javascript
    function activateAdminUser(user) { ... }
    function activateStandardUser(user) { ... }
    ```

---

## 10. Method Length
Avoid long methods (more than 50 lines).

---

## 11. Class Length
Avoid long classes (more than 300 lines).

---

## 12. Blank Lines inside Methods
Avoid blank lines inside methods and functions.

*   **Before:**
    ```javascript
    function calculate() {
      const a = 1;

      const b = 2;
      return a + b;
    }
    ```
*   **After:**
    ```javascript
    function calculate() {
      const a = 1;
      const b = 2;
      return a + b;
    }
    ```

---

## 13. Comments
Avoid using comments whenever possible; make the code self-explanatory.

*   **Before:** `const d = 10; // days until expiration`
*   **After:** `const daysUntilExpiration = 10;`

---

## 14. Multiple Variables per Line
Never declare more than one variable on the same line.

*   **Before:** `let x = 1, y = 2;`
*   **After:**
    ```javascript
    let x = 1;
    let y = 2;
    ```

---

## 15. Variable Proximity
Declare variables as close as possible to where they will be used.

*   **Before:**
    ```javascript
    function process() {
      const total = 0;
      // ... 20 lines of code ...
      return total + 10;
    }
    ```
*   **After:**
    ```javascript
    function process() {
      // ... 20 lines of code ...
      const total = 0;
      return total + 10;
    }
    ```
