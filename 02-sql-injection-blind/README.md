# SQL Injection (Blind)

**Status:** Completed

## Objective

Investigate blind SQL injection in DVWA and document the testing process.

## What is Blind SQL Injection?

Blind SQL Injection occurs when an application is vulnerable to SQL injection, but the application does not directly display the result of the injected query.

Instead, the application's response can be used as a true/false indicator.

In this lab, a result is returned when the SQL condition is true, while no result is returned when the condition is false.

---

## 1. Normal Behaviour

The first step was to test the application with a normal User ID.

**Input:** `1`

**Result:** The application returned:

`First name: admin`  
`Surname: admin`

This established the normal behaviour of the application.

**Evidence:** `01-normal-input.png`

---

## 2. Confirming Blind SQL Injection

The next step was to determine whether the application's response changed depending on a SQL condition.

### True condition

**Input:** `1' AND 1=1 #`

The application returned user information.

This indicates that the condition evaluated to true.

**Evidence:** `02-boolean-true.png`

### False condition

**Input:** `1' AND 1=2 #`

The application returned no user information.

This indicates that the condition evaluated to false.

**Evidence:** `03-boolean-false.png`

### Conclusion

The difference between the true and false conditions confirmed that the application can be used as a boolean oracle.

This successfully demonstrated Boolean-based Blind SQL Injection.

---

## 3. Source Code Analysis

The source code revealed that the User ID is inserted directly into the SQL query.

The vulnerable query is:

`$getid = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";`

The query is then executed using:

`$result = mysql_query($getid);`

The source also uses error suppression when executing the query, which contributes to the blind behaviour of the application.

**Evidence:** `04-source-code.png`

### Root Cause

The main root cause is the direct concatenation of user-controlled input into an SQL query without using parameterized queries.

---

## 4. Database Enumeration

After confirming the vulnerability, boolean-based queries were used to investigate information about the database.

### Number of users

The following conditions were tested:

- `COUNT(*) > 0`
- `COUNT(*) > 1`
- `COUNT(*) > 2`
- `COUNT(*) > 3`
- `COUNT(*) > 4`
- `COUNT(*) > 5`

The conditions up to `COUNT(*) > 4` returned a result, while `COUNT(*) > 5` returned no result.

### Finding

The `users` table contains:

**5 users**

---

## 5. User and Column Verification

The same blind SQL injection technique was used to verify specific information in the database.

### Admin user

A boolean condition confirmed that a user with:

`first_name = admin`

exists.

A separate test confirmed:

`last_name = admin`

The returned data matched the normal application behaviour.

### Password column

Database metadata was queried to determine whether the `users` table contains a column named:

`password`

### Finding

The `users` table contains a `password` column.

---

## 6. Database Identification

Boolean-based queries were also used to identify the database name by testing prefixes.

Examples included:

- `d%`
- `dv%`
- `dvw%`

The application returned a result for these conditions.

This demonstrated how database information can be inferred even when the application does not directly reveal the queried value.

---

## 7. Password Field Analysis

The length of the `admin` password value was investigated using boolean conditions.

The following tests were performed:

- `LENGTH(password) > 10` → True
- `LENGTH(password) > 20` → True
- `LENGTH(password) > 30` → True
- `LENGTH(password) > 40` → False
- `LENGTH(password) > 35` → False
- `LENGTH(password) > 32` → False
- `LENGTH(password) > 31` → True

### Finding

The `admin` password value has a length of:

**32 characters**

The actual password value is not included in this repository.

**Evidence:** `05-password-length.png`

---

## 8. Findings

The Blind SQL Injection vulnerability allowed us to:

- Confirm SQL injection using boolean conditions
- Use the application's response as a true/false indicator
- Determine that the `users` table contains 5 users
- Verify the existence of the `admin` user
- Verify the existence of the `password` column
- Partially identify the database name
- Determine that the `admin` password value is 32 characters long

This demonstrates that database information can be inferred even when the application does not directly display the queried SQL results.

---

## 9. Impact

An attacker exploiting this vulnerability could potentially infer information from the database through repeated boolean queries.

Depending on database permissions and application configuration, this could expose:

- Database structure
- User information
- Sensitive application data
- Password-related information

The practical impact depends on the privileges available to the application's database account.

---

## 10. Remediation

The vulnerability should be fixed by preventing user input from being concatenated directly into SQL queries.

### Recommended approach

Use prepared statements or parameterized queries.

Instead of directly concatenating user input into the SQL statement, the application should use a parameterized query where the user input is supplied separately from the SQL statement.

Additional protections should include:

- Input validation
- Least-privilege database accounts
- Proper error handling
- Avoid exposing SQL/database errors to users

---

## 11. Evidence

The following screenshots document the investigation:

| File | Description |
|---|---|
| `01-normal-input.png` | Normal application behaviour |
| `02-boolean-true.png` | True SQL condition returns a result |
| `03-boolean-false.png` | False SQL condition returns no result |
| `04-source-code.png` | Vulnerable SQL query in source code |
| `05-password-length.png` | Tests demonstrating the password length |

---

## Conclusion

The DVWA application was successfully demonstrated to be vulnerable to Boolean-based Blind SQL Injection.

The vulnerability exists because user-controlled input is directly incorporated into an SQL query.

Although the application does not directly expose the result of arbitrary queries, differences in its responses can be used to infer information from the database.

This lab demonstrates why SQL queries should be implemented using parameterized statements rather than directly concatenating user input.
