---
inclusion: always
---

# Coding Preference
- Write code in Python.

## Python Code Style
- Follow PEP 8, use Black formatting (88 chars)
- Type hints required for all functions

## Python Frameworks
When creating Python code, use the following guidance:

- Use Flask as the web framework
- Follow Flask's application factory pattern
- Use Pydantic for data validation
- Use environment variables for configuration
- Implement Flask-SQLAlchemy for database operations

## Python Project Structure and Layout
- Use __init__.py for packages
- Use the following project structure

```
├ app
	├── src
	├── src/static/
	├── src/models/
	├── src/routes/
	├── src/templates/
	├── src/extensions.py
    ├── tests
    ├── docs
```

## Common Patterns
- Use pathlib.Path for file operations
- Use logging instead of print()

## Python Error Handling
- Use specific exceptions, avoid bare except:

## Python Testing
- Use pytest with fixtures for setup/teardown
- Configure a temporary, ephemeral database every time the tests are run - do NOT use the existing local database which might have data
- Structure tests to mirror source code
- Use descriptive function names starting with `test_`
- Prefer fixtures over setup/teardown methods
- Use assert statements directly, not self.assertEqual


# Python Package Management with uv
- MUST use virtual environment before installing libraries
- All Python dependencies **must be installed, synchronized, and locked** using uv
- Never use pip, pip-tools, poetry, or conda directly for dependency management

## Python Package Management Commands
Use these commands
- Install dependencies: `uv add <package>`
- Remove dependencies: `uv remove <package>`
- Sync dependencies: `uv sync`

## Running Python Code
- Run a Python script with `uv run <script-name>.py`
- Run Python tools like Pytest with `uv run pytest` or `uv run ruff`
- Launch a Python repl with `uv run python`