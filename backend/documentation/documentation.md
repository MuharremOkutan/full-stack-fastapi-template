# FastAPI Backend Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Project Structure](#project-structure)
4. [Dependencies](#dependencies)
5. [Core Components](#core-components)
   - [Configuration](#configuration)
   - [Database](#database)
   - [Security](#security)
   - [Models](#models)
   - [API Endpoints](#api-endpoints)
6. [Development Guide](#development-guide)
   - [Setup Environment](#setup-environment)
   - [Making Changes](#making-changes)
   - [Database Migrations](#database-migrations)
   - [Testing](#testing)
   - [Email Templates](#email-templates)
7. [Deployment](#deployment)

## Introduction

This backend application is built using FastAPI, a modern, high-performance web framework for building APIs with Python. The project follows a modular design pattern with clear separation of concerns, making it maintainable and scalable.

## Architecture Overview

The application follows a layered architecture:

1. **API Layer**: Handles HTTP requests/responses, routing, and input validation using FastAPI.
2. **Service Layer**: Contains business logic and orchestrates data flow between API and data layers.
3. **Data Layer**: Manages database interactions using SQLModel and SQLAlchemy.

The backend implements a RESTful API with JWT-based authentication, PostgreSQL database, and includes features like:
- User management (registration, authentication, profile updates)
- Item management (CRUD operations)
- Email notifications
- Error tracking with Sentry

## Project Structure

```
backend/
├── app/                      # Application code
│   ├── alembic/              # Database migration scripts
│   ├── api/                  # API endpoints and dependencies
│   │   ├── routes/           # API route handlers
│   │   ├── deps.py           # Dependency injection utilities
│   │   └── main.py           # API router configuration
│   ├── core/                 # Core application components
│   │   ├── config.py         # Application settings
│   │   ├── db.py             # Database connection
│   │   └── security.py       # Authentication and security utilities
│   ├── email-templates/      # Email templates (MJML and HTML)
│   ├── tests/                # Unit and integration tests
│   ├── models.py             # Data models (SQLModel)
│   ├── crud.py               # CRUD operations
│   ├── utils.py              # Utility functions
│   └── main.py               # Application entry point
├── scripts/                  # Helper scripts
├── Dockerfile                # Docker configuration
├── requirements.txt          # Python dependencies
├── pyproject.toml            # Project configuration
└── alembic.ini               # Alembic configuration
```

## Dependencies

The project relies on the following key dependencies:

- **FastAPI**: Modern web framework for building APIs
- **Pydantic**: Data validation and settings management
- **SQLModel**: ORM for interacting with databases combining SQLAlchemy and Pydantic
- **Alembic**: Database migration tool
- **Passlib**: Password hashing library
- **PyJWT**: JWT token handling
- **Tenacity**: Retry library for improved resilience
- **Psycopg**: PostgreSQL adapter
- **Emails/Jinja2**: Email templating and sending
- **Sentry SDK**: Error tracking and monitoring

Development dependencies include:
- **Pytest**: Testing framework
- **Mypy**: Static type checking
- **Ruff**: Fast Python linter
- **pre-commit**: Git hooks manager

For a complete list of dependencies with their versions, refer to the `requirements.txt` and `pyproject.toml` files.

## Core Components

### Configuration

The application uses a Pydantic-based settings management system (`app/core/config.py`) that supports:
- Environment variable loading
- Type validation
- Secret validation
- Computed fields
- Environment-specific settings (local, staging, production)

Key configuration options include database connection, JWT settings, SMTP configuration, and CORS settings.

### Database

The database layer is implemented using SQLModel, which combines SQLAlchemy Core with Pydantic models:
- Connection setup and session management in `app/core/db.py`
- Migrations managed by Alembic 
- Models defined in `app/models.py`

The application uses PostgreSQL as its database system.

### Security

Security features include:
- JWT-based authentication with token expiration
- Password hashing with bcrypt
- Role-based access control (superuser/normal user)
- CORS protection
- Dependency-based permission checks

### Models

Models are defined using SQLModel, which combines SQLAlchemy's ORM capabilities with Pydantic's validation:

Key models include:
- **User**: User account information with relationships
- **Item**: Example resource owned by users
- **Various schema models**: For API request/response validation

Models handle both database schema definition and API validation.

### API Endpoints

API endpoints are organized in the `app/api/routes` directory:
- `login.py`: Authentication endpoints
- `users.py`: User management
- `items.py`: Item CRUD operations
- `private.py`: Protected endpoints example
- `utils.py`: Utility endpoints

The API uses dependency injection for:
- Database sessions
- Current user resolution
- Permission checks
- Input validation

## Development Guide

### Setup Environment

#### Option 1: Using Docker Compose (Recommended)

1. Ensure Docker and Docker Compose are installed
2. Start the development environment:
   ```
   docker compose watch
   ```

This creates a development environment with hot-reloading enabled.

#### Option 2: Using uv (Local Development)

1. Install uv: https://docs.astral.sh/uv/
2. From `./backend/` directory, install dependencies:
   ```
   uv sync
   ```
3. Activate the virtual environment:
   ```
   source .venv/bin/activate
   ```

#### Option 3: Using venv and pip

1. Create a virtual environment:
   ```
   python -m venv venv
   ```
2. Activate the virtual environment:
   ```
   # On Linux/macOS
   source venv/bin/activate
   
   # On Windows
   venv\Scripts\activate
   ```
3. Install dependencies:
   ```
   pip install -r requirements.txt
   ```

### Making Changes

#### Modifying Data Models

1. Update model definitions in `app/models.py`
2. Create a database migration (see Database Migrations section)
3. Update corresponding API endpoints and CRUD operations

#### Adding New API Endpoints

1. Create or modify route handlers in `app/api/routes/`
2. Register routes in `app/api/main.py` if needed
3. Implement required business logic
4. Add appropriate tests

#### Adding New Dependencies

1. Add the dependency to `requirements.txt` or `pyproject.toml`
2. Install with `uv sync` or `pip install -r requirements.txt`
3. Import and use the dependency in your code

### Database Migrations

Database changes are managed using Alembic:

1. Make changes to your models in `app/models.py`
2. Generate a migration:
   ```
   # In Docker
   docker compose exec backend bash
   alembic revision --autogenerate -m "Description of changes"
   ```
3. Apply the migration:
   ```
   alembic upgrade head
   ```
4. Commit the generated migration files to version control

To roll back a migration:
```
alembic downgrade <target_revision>
```

### Testing

Tests are located in the `app/tests/` directory. To run tests:

```
# Using the script
bash ./scripts/test.sh

# Or with Docker
docker compose exec backend bash scripts/tests-start.sh
```

Test coverage reports are generated in the `htmlcov/` directory.

To add new tests:
1. Create test files in `app/tests/`
2. Follow the existing test patterns
3. Run tests to ensure they pass

### Email Templates

Email templates use the MJML framework to create responsive email designs:

1. Source files are in `app/email-templates/src/` in MJML format
2. Compiled HTML templates are in `app/email-templates/build/`

To create or modify templates:
1. Install the MJML VS Code extension
2. Edit the MJML file in the `src` directory
3. Use the MJML extension to export to HTML
4. Save the HTML file in the `build` directory

## Deployment

The application is containerized with Docker, making deployment straightforward:

1. Configure environment variables in a `.env` file or directly in your deployment platform
2. Build the Docker image:
   ```
   docker build -t myapp-backend .
   ```
3. Run the container with appropriate environment variables:
   ```
   docker run -p 8000:80 --env-file .env myapp-backend
   ```

For production deployments:
- Use a proper reverse proxy (Nginx, Traefik)
- Set up proper TLS/SSL
- Configure database backups
- Set up monitoring and logging
- Use container orchestration for high availability
