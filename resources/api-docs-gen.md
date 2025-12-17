# Task: Generate OpenAPI 3 Documentation for Python Flask Projects

You are tasked with analyzing a Python Flask codebase and generating comprehensive OpenAPI documentation.

### Step 1: Analyze Each Flask Route

For each Flask API route, carefully analyze:

- **HTTP Methods**: Extract from Flask route decorators (`@app.route`, `@bp.route`) with methods parameter
- **Path Parameters**: Extract from Flask route patterns (e.g., `/users/<int:user_id>`, `/posts/<string:slug>`)
- **Request Body**: Analyze Flask request handling (`request.json`, `request.form`, Pydantic models)
- **Response Types**: Analyze Flask response patterns (`jsonify()`, `make_response()`, return tuples)
- **Status Codes**: Extract from Flask response tuples, `abort()` calls, and explicit status codes
- **Authentication**: Look for Flask decorators (`@login_required`, `@jwt_required`, custom auth decorators)
- **Error Handling**: Identify Flask error handlers (`@app.errorhandler`) and custom exceptions
- **Type Hints**: Leverage Python type hints and Pydantic models for schema inference
- **Docstrings**: Extract information from function docstrings following Python conventions

### Step 2: Review Existing Documentation

**IMPORTANT**: Documentation files may already exist from previous generation runs.

Before making any changes:
1. List all existing files in the current project workspace.
3. Read each existing file to understand what's already documented.
4. Compare existing documentation with current API route implementations.

### Step 3: Generate or Update Documentation Files

#### A. Schema Files

Create or update schema files for all data structures. Group related schemas by entity.

**Update Strategy**:
- If a schema file exists: Read it first, then update only if changes are needed.
- If a schema file is missing: Create a new file.
- If a schema is no longer used: Delete the schema definition from the file.

**File naming**: Use entity name (e.g., `user.json`, `survey.json`, `error.json`)

**Python-Specific Schema Inference**:
- **SQLAlchemy Models**: Extract field types, constraints, and relationships from model definitions
- **Pydantic Models**: Use Pydantic schema generation and validation rules
- **Type Hints**: Leverage Python type annotations (`Optional`, `List`, `Dict`, `Union`)
- **Flask-WTF Forms**: Extract validation rules from WTForms field definitions
- **Dataclasses**: Use dataclass field definitions and default values
- Add `"nullable": true` for `Optional` types and fields with `default=None`
- Include `description` from docstrings and field comments
- Use JSON Schema formats: `date-time`, `email`, `uri`, etc.
- Add validation constraints from Pydantic validators, SQLAlchemy constraints, and WTForms validators
- Handle Python-specific types: `datetime`, `UUID`, `Decimal`, `Enum`

#### B. Route Files

Create or update one file per API endpoint with 1-1 mapping to Flask route functions.

**Update Strategy**:
- If a route file exists: Read it first, then update only if changes are needed
- If a route file is missing: Create a new file
- If a route file exists but the Flask route was deleted: Delete the route file

**Required Fields**:
- `summary` - Extract from function name or first line of docstring
- `description` - Extract from function docstring (Google/Sphinx style)
- `operationId` - Generate from function name (snake_case to camelCase conversion)
- `tags` - Use Flask Blueprint names or route prefixes for categorization
- `security` - Detect from Flask decorators (`@login_required`, `@jwt_required`, etc.)
- `responses` - Include all possible status codes from Flask responses and error handlers

**Flask-Specific Considerations**:
- Handle Flask route parameter types (`<int:id>`, `<string:name>`, `<path:filename>`)
- Extract request validation from Pydantic models or WTForms
- Document Flask-specific response patterns (`jsonify`, `render_template`, `redirect`)
- Include Flask error handling (`abort()`, custom error handlers)
- Handle Flask request context (`request.args`, `request.json`, `request.files`)

### Step 4: Validation

Before writing/updating each file:

1. Ensure valid JSON syntax.
2. Verify all `$ref` references exist.
3. Check that paths are correctly formatted.
4. Confirm schema definitions are complete.
5. Only write if changes are needed (don't rewrite identical content).

## Execution Instructions

1. **Inventory Phase**:

   - List all API routes (endpoints).
   - List all existing schema files.
   - List all existing route files.

2. **Analysis Phase**:

   - For each API route, analyze its implementation (methods, params, types, errors).
   - Read corresponding existing documentation files.
   - Identify differences between code and documentation.

3. **Update Phase**:

   - Update/create schema files only if changes are needed.
   - Update/create route files only if changes are needed.
   - Delete route files for API routes that no longer exist.

4. **Verification Phase**:

   - Use `uv run python -c "import json; json.load(open('openapi.json'))"` to validate JSON syntax of the generated spec.
   - Use `uv add openapi-spec-validator` (if not already installed) for OpenAPI validation.
   - Run `uv run python -c "from openapi_spec_validator import validate_spec; import json; validate_spec(json.load(open('openapi.json')))"` to validate the OpenAPI spec.
   - Alternatively, use `uv add pydantic[email]` and create a simple validation script using Pydantic models.
   - If validation fails:
     - Read the error messages carefully.
     - Identify which files have issues.
     - Fix the problems in the schema/route files.
     - Repeat this verification phase.
   - Maximum 5 verification attempts allowed.
   - If still failing after 5 attempts, report the errors and exit.

5. **Summary Phase**:

   - List all files created.
   - List all files updated.
   - List all files deleted.
   - Report final validation status.
   - Confirm all changes are complete.