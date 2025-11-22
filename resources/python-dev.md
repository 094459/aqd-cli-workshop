## Directory Structure

- Use the following project structure

```
project/
├── src/
├── tests/
├── .env.example
└── run.py (entry point)
```

## Architecting guidelines

- Use modular design.
- Follow Single Responsibility Principle.
- Follow DRY (Don't Repeat Yourself) Principle.

## Development Guidelines

- Use uv for managing Python dependencies and packages
- Use virtual environments (venv)
- Use proper Git workflow
- Follow semantic versioning
- Pin dependency versions

---

# Code Style

## Python

- Always use UV when installing dependencies.
- Always use Python 3.12.
- Use Black for code formatting.
- Use Pylint for linting.
- Follow PEP 8 and project-specific rules.
- Indent: 4 spaces
- Maximum line length of 88 characters (Black default)
- Docstring: Google style
- Use absolute imports over relative imports

## Configuration management

- Use .env files for configuration.
- Use python-dotenv for environment variable management.
- Manage secrets using environment variables.
- Never commit secrets to version control

## Code Structure

## Naming Convention

- snake_case for functions and variables
- PascalCase for classes
- UPPER_CASE for constants

## Function Guidelines

- Maximum function length: 50 lines
- Maximum nesting depth: 3 levels
- Single responsibility principle
- Clear return types 

## Documentation

- Provide detailed documentation using docstrings and README files.

## Handling Errors

- Prefer using try-except blocks for error handling.
- Log errors appropriately.

## Logging

- Use the Python logging module for logging.
- Use log levels: debug, info, warn, error.
- Set a log retention policy of 7 days.

## Security

- Require HTTPS for secure connections.
- Sanitize all inputs.
- Use environment variables for sensitive configuration.
- Implement CSRF protection with Flask-WTF
- Input validation with Marshmallow/Pydantic
- Implement SQL injection prevention
- Ensure XSS protection
- Never commit secrets to version control

## Performance and Optimisation

- Imeplement Database query optimization
- For functions that require low latency, implement caching strategies (Redis/Memcached)
- Implement Connection pooling


# Testing Standards

- Use PyTest with Flask-Testing
- Create fixtures in conftest.py
- Use PyTest.ini
- Test structure: tests/unit/, tests/integration/

