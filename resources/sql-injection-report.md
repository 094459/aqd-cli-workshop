# SQL Injection Protection Analysis - export_survey Function

## Executive Summary

**The `export_survey` function is PROTECTED against SQL injection attacks.**

The protection comes from multiple layers of defense, primarily through SQLAlchemy's ORM and parameterized queries.

---

## Layer-by-Layer Protection Analysis

### **Layer 1: Flask Route Type Coercion**

```python
@surveys_bp.route("/survey/<int:survey_id>/export/<format>")
def export_survey(survey_id: int, format: str):
```

**Protection Mechanism**:
- `<int:survey_id>` - Flask's route converter ensures `survey_id` is an integer
- If user sends `/survey/1' OR '1'='1/export/csv`, Flask returns 404 (route not found)
- Non-integer values never reach the function

**Attack Blocked**:
```
GET /survey/1'; DROP TABLE users; --/export/csv
→ Flask: 404 Not Found (route doesn't match)
```

**Limitation**: The `format` parameter is a string and not validated by route, but it's validated in the function.

---

### **Layer 2: SQLAlchemy ORM Parameterization**

```python
survey = Survey.query.filter_by(id=survey_id, user_id=current_user.id).first()
```

**What SQLAlchemy Generates**:
```sql
SELECT * FROM surveys 
WHERE survey_id = ? AND user_id = ?
```

**Protection Mechanism**:
- SQLAlchemy uses **parameterized queries** (prepared statements)
- `survey_id` and `user_id` are passed as parameters, NOT concatenated into SQL string
- Database driver treats parameters as data, never as SQL code

**Attack Attempt**:
```python
# Attacker somehow bypasses Flask routing and passes:
survey_id = "1 OR 1=1"

# SQLAlchemy generates:
SELECT * FROM surveys WHERE survey_id = ? AND user_id = ?
# With parameters: ["1 OR 1=1", 42]

# Database tries to convert "1 OR 1=1" to integer → fails
# Query returns no results (or raises error)
```

**Why This Works**:
- Parameters are sent separately from SQL statement
- Database knows "this is data, not code"
- No string concatenation = no injection point

---

### **Layer 3: ORM Query Builder (No String Concatenation)**

```python
responses = db.session.query(
    SurveyResponse.id,
    SurveyResponse.response_date,
    SurveyResponse.respondent_email,
    SurveyOption.option_text
).join(
    SurveyOption, SurveyResponse.option_id == SurveyOption.id
).filter(
    SurveyResponse.survey_id == survey_id
).order_by(
    SurveyResponse.response_date
).all()
```

**Generated SQL** (approximate):
```sql
SELECT 
    survey_responses.response_id,
    survey_responses.response_date,
    survey_responses.respondent_email,
    survey_options.option_text
FROM survey_responses
JOIN survey_options ON survey_responses.option_id = survey_options.option_id
WHERE survey_responses.survey_id = ?
ORDER BY survey_responses.response_date
```

**Protection Mechanism**:
- Query built using method chaining, not string concatenation
- All user inputs passed as parameters
- SQLAlchemy escapes/parameterizes everything

**What Would Be Vulnerable** (NOT in this code):
```python
# DANGEROUS - DO NOT DO THIS
query = f"SELECT * FROM responses WHERE survey_id = {survey_id}"
db.session.execute(query)
```

---

### **Layer 4: Format Validation (Defense in Depth)**

```python
if format not in ["csv", "json"]:
    return "Invalid format", 400
```

**Protection Mechanism**:
- Whitelist validation - only allows exact matches
- `format` parameter never touches database
- Even if attacker sends `format="csv'; DROP TABLE users; --"`, it's rejected before any DB query

**Attack Blocked**:
```
GET /survey/1/export/csv'; DROP TABLE users; --
→ format = "csv'; DROP TABLE users; --"
→ Not in ["csv", "json"]
→ Returns 400 Bad Request
```

---

## Attack Scenarios & Why They Fail

### **Scenario 1: Classic SQL Injection in survey_id**

**Attack**:
```
GET /survey/1' OR '1'='1/export/csv
```

**Defense**:
1. Flask routing fails (not an integer) → 404
2. Even if it reached the function, SQLAlchemy parameterizes it

**Result**: ❌ Attack fails at routing layer

---

### **Scenario 2: Second-Order SQL Injection via Data**

**Attack**:
1. Attacker creates survey with title: `Test'; DROP TABLE users; --`
2. Exports that survey
3. Hopes the title gets executed as SQL

**What Actually Happens**:
```python
data = {
    "title": survey.title,  # Contains: "Test'; DROP TABLE users; --"
}
```

**Defense**:
- `survey.title` is just data retrieved from database
- It's never used in a SQL query
- It's output to JSON/CSV as a string
- CSV writer escapes special characters automatically

**Result**: ❌ Attack fails - data is treated as data, not code

---

### **Scenario 3: Injection via respondent_email**

**Attack**:
1. Attacker submits survey response with email: `test@example.com'; DROP TABLE users; --`
2. Owner exports survey
3. Hopes email gets executed as SQL

**What Actually Happens**:
```python
writer.writerow([r.id, r.response_date, r.respondent_email or "", r.option_text])
```

**Defense**:
- `respondent_email` retrieved via parameterized query
- CSV writer escapes it: `"test@example.com'; DROP TABLE users; --"`
- Never used in subsequent SQL queries

**Result**: ❌ Attack fails - CSV escaping prevents code execution

---

### **Scenario 4: Injection via option_text**

**Attack**:
1. Attacker creates survey option: `Option 1'; DELETE FROM surveys WHERE '1'='1`
2. Exports survey
3. Hopes option text gets executed

**Defense**:
- Option text retrieved via JOIN with parameterized query
- Output to CSV/JSON as escaped string
- Never used in SQL query construction

**Result**: ❌ Attack fails

---

## What WOULD Be Vulnerable (Examples of Bad Code)

### **Vulnerable Pattern 1: String Formatting**
```python
# DANGEROUS - NOT IN YOUR CODE
query = f"SELECT * FROM responses WHERE survey_id = {survey_id}"
db.session.execute(query)
```

### **Vulnerable Pattern 2: Raw SQL with Concatenation**
```python
# DANGEROUS - NOT IN YOUR CODE
sql = "SELECT * FROM responses WHERE survey_id = " + str(survey_id)
db.session.execute(sql)
```

### **Vulnerable Pattern 3: Using User Input in ORDER BY**
```python
# DANGEROUS - NOT IN YOUR CODE
order_by_column = request.args.get('sort')  # User provides: "id; DROP TABLE users"
query = f"SELECT * FROM responses ORDER BY {order_by_column}"
```

---

## Why Your Code is Safe

### ✅ **Uses ORM Query Builder**
- No string concatenation
- All inputs parameterized

### ✅ **Type Coercion at Route Level**
- Flask ensures `survey_id` is integer
- Invalid types rejected before reaching function

### ✅ **Whitelist Validation**
- `format` parameter validated against allowed values
- Rejects anything not in whitelist

### ✅ **No Dynamic SQL Construction**
- No f-strings or string concatenation in queries
- No raw SQL execution with user input

### ✅ **Defense in Depth**
- Multiple layers of protection
- Even if one layer fails, others protect

---

## Potential Weaknesses (Not SQL Injection, but Related)

### **1. CSV Injection (Formula Injection)**

**Current Code**:
```python
writer.writerow([r.id, r.response_date, r.respondent_email or "", r.option_text])
```

**Risk**: If `option_text` starts with `=`, `+`, `-`, or `@`, Excel might interpret it as a formula.

**Example Attack**:
- Option text: `=1+1` → Excel executes formula
- Option text: `=cmd|'/c calc'!A1` → Could execute commands in Excel

**Mitigation**:
```python
def sanitize_csv_field(value):
    """Prevent CSV injection by prefixing dangerous characters."""
    if isinstance(value, str) and value and value[0] in ('=', '+', '-', '@'):
        return "'" + value  # Prefix with single quote
    return value

writer.writerow([
    r.id, 
    r.response_date, 
    sanitize_csv_field(r.respondent_email or ""), 
    sanitize_csv_field(r.option_text)
])
```

### **2. HTTP Header Injection (Mentioned in Review)**

**Current Code**:
```python
headers={"Content-Disposition": f"attachment;filename=survey_{survey_id}_responses.csv"}
```

**Risk**: If `survey_id` contained newlines (impossible due to `<int:` converter), could inject headers.

**Current Protection**: Flask's `<int:survey_id>` prevents this.

**Best Practice Enhancement**:
```python
from werkzeug.utils import secure_filename
safe_filename = secure_filename(f"survey_{survey_id}_responses.csv")
headers={"Content-Disposition": f"attachment;filename={safe_filename}"}
```

---

## Conclusion

**SQL Injection Protection: ✅ EXCELLENT**

The `export_survey` function is **well-protected** against SQL injection through:

1. **Flask route type validation** - Rejects non-integer survey_ids
2. **SQLAlchemy ORM** - Uses parameterized queries exclusively
3. **No string concatenation** - All queries built with query builder
4. **Input validation** - Format parameter whitelisted

**Risk Level**: **LOW** for SQL injection

**Recommendation**: The SQL injection protection is solid. Focus security efforts on:
- Rate limiting (prevent abuse)
- CSV injection mitigation (if users open exports in Excel)
- Input length validation (prevent DoS)

The code follows security best practices for database interaction.
