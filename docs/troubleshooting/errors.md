---
title: Troubleshooting
nav_order: 99
permalink: /troubleshooting/errors/
---

# Troubleshooting

Quick fixes for common JoinQuant errors:

---

### 🔹 SyntaxError
- **Cause**: Missing comma / wrong brackets
- **Fix**: Check commas in `query()` and indentation

---

### 🔹 AttributeError
- **Cause**: Wrong field name (e.g. `valuation,pe_ratio`)
- **Fix**: Use `valuation.pe_ratio` and confirm fields exist

---

### 🔹 Date Error
- **Cause**: Using `data='2019-09-01'`
- **Fix**: Correct param is `date='YYYY-MM-DD'`

---

📌 Tip: Always `print(df.head())` to check returned columns
