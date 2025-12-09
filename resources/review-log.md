# Code Review - Customer Survey Application

**Review Date**: 2025-12-09  
**Reviewer**: Rovie  
**Scope**: Security, Code Quality, Architecture, Best Practices

---

## 🔴 CRITICAL ISSUES

### 1. **Weak Secret Key in Production**
**File**: `src/__init__.py:26`
```python
app.config["SECRET_KEY"] = os.getenv("SECRET_KEY", "dev-secret-key")
```
**Problem**: Falls back to hardcoded "dev-secret-key" if environment variable not set. This key signs session cookies - if compromised, attackers can forge sessions and impersonate any user.

**Impact**: Complete authentication bypass in production if SECRET_KEY not set.

**Fix**:
```python
secret_key = os.getenv("SECRET_KEY")
if not secret_key:
    raise RuntimeError("SECRET_KEY environment variable must be set")
app.config["SECRET_KEY"] = secret_key
```

---

### 2. **Deprecated datetime.utcnow() Usage**
**Files**: Multiple locations
- `src/models/user.py:18`
- `src/models/survey.py:17-18`
- `src/routes/auth.py:56`

**Problem**: `datetime.utcnow()` is deprecated in Python 3.12+ and will be removed. Creates naive datetime objects that can cause timezone bugs.

**Impact**: Code will break in future Python versions. Timezone-related bugs.

**Fix**:
```python
from datetime import datetime, timezone

# Replace all instances of:
datetime.utcnow()
# With:
datetime.now(timezone.utc)
```

---

## 🟠 HIGH PRIORITY ISSUES

### 3. **No Password Strength Validation**
**File**: `src/routes/auth.py:23`
```python
password = request.form.get("password", "")
if not email or not password:
    flash("Email and password are required")
```
**Problem**: Accepts any password, including "1", "a", empty strings after strip.

**Impact**: Weak passwords make accounts vulnerable to brute force attacks.

**Fix**:
```python
if len(password) < 8:
    flash("Password must be at least 8 characters")
    return render_template("register.html")
```

---

### 4. **No Email Validation**
**File**: `src/routes/auth.py:22`
```python
email = request.form.get("email", "").strip()
```
**Problem**: Accepts invalid emails like "notanemail", "test@", "@example.com".

**Impact**: Database pollution, broken email functionality if added later.

**Fix**:
```python
import re
EMAIL_REGEX = re.compile(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')

if not EMAIL_REGEX.match(email):
    flash("Invalid email address")
    return render_template("register.html")
```

---

### 5. **Missing Rate Limiting on Authentication**
**File**: `src/routes/auth.py:44-62`

**Problem**: No rate limiting on login attempts. Attacker can brute force passwords.

**Impact**: Account takeover via credential stuffing or brute force.

**Fix**:
```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@auth_bp.route("/login", methods=["GET", "POST"])
@limiter.limit("5 per minute")
def login():
    # ... existing code
```

---

### 6. **No CSRF Protection on Public Survey Response**
**File**: `src/routes/surveys.py:87`
```python
@surveys_bp.route("/s/<int:survey_id>", methods=["GET", "POST"])
def survey_response(survey_id: int) -> str | tuple[str, int]:
```
**Problem**: Public endpoint accepts POST without CSRF token. Attacker can submit fake responses via CSRF.

**Impact**: Survey data poisoning, fake responses.

**Fix**:
```python
from flask_wtf.csrf import CSRFProtect

# In template survey_response.html, add:
# <input type="hidden" name="csrf_token" value="{{ csrf_token() }}"/>

# Or exempt if truly public:
@csrf.exempt
@surveys_bp.route("/s/<int:survey_id>", methods=["GET", "POST"])
```

---

### 7. **Information Disclosure in Login**
**File**: `src/routes/auth.py:58-61`
```python
if user and check_password_hash(user.password_hash, password):
    # ... login
else:
    flash("Invalid email or password")
```
**Problem**: Same error message for wrong email vs wrong password is good. But timing attack possible - checking password hash takes longer than checking if user exists.

**Impact**: Attacker can enumerate valid email addresses via timing analysis.

**Fix**:
```python
user = User.query.filter_by(email=email).first()

# Always check password hash, even if user doesn't exist
dummy_hash = generate_password_hash("dummy")
password_hash = user.password_hash if user else dummy_hash
is_valid = check_password_hash(password_hash, password)

if user and is_valid:
    # ... login
else:
    flash("Invalid email or password")
```

---

## 🟡 MEDIUM PRIORITY ISSUES

### 8. **No Input Length Validation**
**File**: `src/routes/surveys.py:30-31`
```python
title = request.form.get("title", "").strip()
description = request.form.get("description", "").strip()
```
**Problem**: No max length check. User could submit 10MB title, causing memory issues or database errors.

**Impact**: DoS via large inputs, database errors.

**Fix**:
```python
MAX_TITLE_LENGTH = 255
MAX_DESCRIPTION_LENGTH = 5000

if len(title) > MAX_TITLE_LENGTH:
    flash(f"Title must be less than {MAX_TITLE_LENGTH} characters")
    return render_template("create_survey.html")
```

---

### 9. **Missing Database Indexes**
**File**: `src/models/survey.py`

**Problem**: No indexes on frequently queried columns:
- `Survey.user_id` (queried in dashboard)
- `SurveyResponse.survey_id` (queried in results/export)
- `SurveyOption.survey_id` (queried everywhere)

**Impact**: Slow queries as data grows. Full table scans.

**Fix**:
```python
class Survey(db.Model):
    # ... existing columns
    user_id = db.Column(db.Integer, db.ForeignKey("users.user_id"), 
                        nullable=False, index=True)

class SurveyResponse(db.Model):
    # ... existing columns
    survey_id = db.Column(db.Integer, db.ForeignKey("surveys.survey_id"), 
                          nullable=False, index=True)
```

---

### 10. **No Logging for Security Events**
**Files**: `src/routes/auth.py`, `src/routes/surveys.py`

**Problem**: No logging for:
- Failed login attempts
- Account creation
- Survey exports
- Unauthorized access attempts

**Impact**: Can't detect attacks, no audit trail, difficult to debug issues.

**Fix**:
```python
import logging

logger = logging.getLogger(__name__)

# In login function:
if not (user and is_valid):
    logger.warning(f"Failed login attempt for email: {email} from IP: {request.remote_addr}")
    
# In export function:
logger.info(f"User {current_user.id} exported survey {survey_id} as {format}")
```

---

### 11. **No Transaction Rollback on Error**
**File**: `src/routes/surveys.py:48-58`
```python
survey = Survey(user_id=current_user.id, title=title, description=description)
db.session.add(survey)
db.session.flush()

for idx, option_text in enumerate(options, 1):
    option = SurveyOption(...)
    db.session.add(option)

db.session.commit()
```
**Problem**: If commit fails after adding options, database could be in inconsistent state. No explicit error handling.

**Impact**: Partial data writes, database corruption.

**Fix**:
```python
try:
    survey = Survey(...)
    db.session.add(survey)
    db.session.flush()
    
    for idx, option_text in enumerate(options, 1):
        option = SurveyOption(...)
        db.session.add(option)
    
    db.session.commit()
except Exception as e:
    db.session.rollback()
    logger.error(f"Failed to create survey: {e}")
    flash("Error creating survey. Please try again.")
    return render_template("create_survey.html")
```

---

## 🟢 LOW PRIORITY / STYLE ISSUES

### 12. **Inconsistent Error Responses**
**File**: `src/routes/surveys.py`

**Problem**: Mix of tuple returns and Response objects:
- Line 104: `return "Survey not found or inactive", 404`
- Line 156: `return "Survey not found", 404`
- Line 159: `return "Invalid format", 400`

**Impact**: Inconsistent API responses, harder to test.

**Suggestion**: Use consistent error handling:
```python
from flask import jsonify

def error_response(message: str, status_code: int):
    return jsonify({"error": message}), status_code

return error_response("Survey not found", 404)
```

---

### 13. **Magic Numbers**
**File**: `src/routes/surveys.py:33`
```python
for i in range(1, 6):  # Why 6?
```
**Problem**: Hardcoded limit of 5 options with no constant.

**Impact**: Hard to change, unclear intent.

**Fix**:
```python
MAX_SURVEY_OPTIONS = 5

for i in range(1, MAX_SURVEY_OPTIONS + 1):
    # ...
```

---

### 14. **Missing Type Hints in Some Functions**
**File**: `src/__init__.py:39`
```python
def load_user(user_id: str):
    # Missing return type hint
```
**Impact**: Reduced IDE support, less clear API.

**Fix**:
```python
from typing import Optional

def load_user(user_id: str) -> Optional[User]:
    from src.models import User
    return User.query.get(int(user_id))
```

---

### 15. **No Database Connection Pooling Configuration**
**File**: `src/__init__.py`

**Problem**: Using default SQLAlchemy pool settings. May not be optimal for production.

**Impact**: Connection exhaustion under load.

**Suggestion**:
```python
app.config["SQLALCHEMY_ENGINE_OPTIONS"] = {
    "pool_size": 10,
    "pool_recycle": 3600,
    "pool_pre_ping": True,  # Verify connections before use
}
```

---

### 16. **Respondent Email Not Validated**
**File**: `src/routes/surveys.py:107`
```python
respondent_email = request.form.get("email", "").strip()
```
**Problem**: Accepts any string as email, including invalid formats.

**Impact**: Bad data in database, can't use for follow-up.

**Fix**: Apply same email validation as registration.

---

## ✅ POSITIVE OBSERVATIONS

1. **Good use of Flask application factory pattern** - Makes testing easier
2. **CSRF protection enabled** - Flask-WTF configured correctly
3. **Password hashing with werkzeug** - Using industry-standard bcrypt
4. **Proper use of Flask-Login** - Authentication handled correctly
5. **Blueprint organization** - Clean separation of concerns
6. **Cascade deletes configured** - Data integrity maintained
7. **Environment variables for config** - Good 12-factor app practice
8. **Comprehensive test coverage (99%)** - Excellent testing discipline
9. **Type hints used consistently** - Improves code quality
10. **.env in .gitignore** - Secrets not committed to repo

---

## 📊 SUMMARY

| Severity | Count | Must Fix Before Production |
|----------|-------|---------------------------|
| Critical | 2 | ✅ Yes |
| High | 5 | ✅ Yes |
| Medium | 6 | ⚠️ Recommended |
| Low | 6 | ❌ Optional |

---

## 🎯 RECOMMENDED ACTION PLAN

### Phase 1 (Before Production Deploy):
1. Fix SECRET_KEY fallback (#1)
2. Add password strength validation (#4)
3. Add rate limiting on auth endpoints (#6)
4. Fix deprecated datetime.utcnow() (#3)
5. Add email validation (#5)

### Phase 2 (Next Sprint):
6. Add security logging (#11)
7. Add database indexes (#10)
8. Add input length validation (#9)
9. Add transaction error handling (#13)

### Phase 3 (Technical Debt):
10. Refactor error responses (#14)
11. Add connection pooling config (#17)
12. Fix timing attack in login (#8)

---

## 🔒 SECURITY SCORE: 6.5/10

**Strengths**: Good authentication foundation, CSRF protection, password hashing  
**Weaknesses**: No rate limiting, weak validation, missing security logging

**Overall Assessment**: Code is well-structured and follows good practices, but has several security gaps that must be addressed before production deployment. The architecture is solid and the codebase is maintainable.
