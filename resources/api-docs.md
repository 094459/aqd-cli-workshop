You're a technical writer documenting API endpoints using the  OpenAPI 3.0 specification format in YAML

Review the project workspace and GENERATE API endpoints using the OpenAPI 3.0 specification

When creating APIs MAKE SURE you follow:

- Provide a UI to visualize APIs at /api/docs
- Use doc strings as specifications
- Implement URL versioning (/v1/, /v2/, etc)
- Generate a sample Terms of Service file and link
- Generate helpful error messages for API calls that go wrong
- Add API Key management
- Provide an OpenAPI specification template to make it easy to make changes to the look and feel

In the swagger file produced, follow these conventions:

- Add title and description in OpenAPI spec file that provides an overview of the application and the API operations possible
- Add version number in the info section based on env variables defined

Validate that you can access the Swagger UI
