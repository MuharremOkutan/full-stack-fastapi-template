# Backend Developer Guide

This guide provides detailed instructions for common development tasks in the FastAPI backend. It covers how to extend the application, create new features, and integrate with the frontend.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Development Environment Setup](#development-environment-setup)
3. [Creating a New Database Model](#creating-a-new-database-model)
4. [Implementing CRUD Operations](#implementing-crud-operations)
5. [Creating API Endpoints](#creating-api-endpoints)
6. [Authentication and Permissions](#authentication-and-permissions)
7. [Working with Database Relationships](#working-with-database-relationships)
8. [Error Handling](#error-handling)
9. [Email Integration](#email-integration)
10. [Testing Your Code](#testing-your-code)
11. [Frontend Integration](#frontend-integration)
12. [Troubleshooting](#troubleshooting)

## Prerequisites

Before you start development, ensure you have:

- Basic understanding of Python and FastAPI
- Familiarity with SQL and ORM concepts
- Docker and Docker Compose installed
- Git installed
- A code editor (VS Code recommended)

## Development Environment Setup

1. Clone the repository and navigate to the project directory:
   ```bash
   git clone <repository-url>
   cd full-stack-fastapi-template
   ```

2. Start the development environment:
   ```bash
   docker compose watch
   ```

3. Access the backend container shell when needed:
   ```bash
   docker compose exec backend bash
   ```

## Creating a New Database Model

This section walks through creating a new database model, adding it to the application, and setting up the necessary migrations.

### Step 1: Define the Model

To define a new model, add it to `app/models.py`. Let's create a `Task` model as an example:

```python
# Task base model for shared properties
class TaskBase(SQLModel):
    title: str = Field(min_length=1, max_length=255)
    description: str | None = Field(default=None, max_length=1000)
    is_completed: bool = Field(default=False)
    due_date: datetime | None = Field(default=None)
    priority: int = Field(default=1, ge=1, le=5)  # Priority from 1-5

# Properties for task creation
class TaskCreate(TaskBase):
    pass

# Properties for task update
class TaskUpdate(TaskBase):
    title: str | None = Field(default=None, min_length=1, max_length=255)  # type: ignore
    is_completed: bool | None = Field(default=None)
    priority: int | None = Field(default=None, ge=1, le=5)

# Database model
class Task(TaskBase, table=True):
    id: uuid.UUID = Field(default_factory=uuid.uuid4, primary_key=True)
    owner_id: uuid.UUID = Field(foreign_key="user.id", nullable=False)
    owner: User | None = Relationship(back_populates="tasks")
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime | None = Field(default=None)

# Public API model
class TaskPublic(TaskBase):
    id: uuid.UUID
    owner_id: uuid.UUID
    created_at: datetime
    updated_at: datetime | None

# List response model
class TasksPublic(SQLModel):
    data: list[TaskPublic]
    count: int
```

### Step 2: Update User Model (for Relationships)

Modify the `User` model to include a relationship to the new `Task` model:

```python
class User(UserBase, table=True):
    id: uuid.UUID = Field(default_factory=uuid.uuid4, primary_key=True)
    hashed_password: str
    items: list["Item"] = Relationship(back_populates="owner")
    tasks: list["Task"] = Relationship(back_populates="owner")  # Add this line
```

### Step 3: Generate and Apply Database Migration

```bash
# Access the backend container
docker compose exec backend bash

# Generate migration
alembic revision --autogenerate -m "Add Task model"

# Apply migration
alembic upgrade head
```

## Implementing CRUD Operations

After creating a model, implement CRUD (Create, Read, Update, Delete) operations in `app/crud.py`.

### Example: CRUD for Tasks

```python
# --- In app/crud.py ---

class CRUDTask(CRUDBase[Task, TaskCreate, TaskUpdate]):
    def create_with_owner(
        self, db: Session, *, obj_in: TaskCreate, owner_id: uuid.UUID
    ) -> Task:
        db_obj = Task(
            **obj_in.model_dump(),
            owner_id=owner_id,
        )
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj

    def get_multi_by_owner(
        self,
        db: Session,
        *,
        owner_id: uuid.UUID,
        skip: int = 0,
        limit: int = 100,
    ) -> list[Task]:
        return (
            db.query(Task)
            .filter(Task.owner_id == owner_id)
            .offset(skip)
            .limit(limit)
            .all()
        )
    
    def update(
        self,
        db: Session,
        *,
        db_obj: Task,
        obj_in: TaskUpdate | dict[str, Any],
    ) -> Task:
        if isinstance(obj_in, dict):
            update_data = obj_in
        else:
            update_data = obj_in.model_dump(exclude_unset=True)
        
        # Add updated_at timestamp
        update_data["updated_at"] = datetime.utcnow()
        
        return super().update(db, db_obj=db_obj, obj_in=update_data)


# At the bottom of the file, instantiate the CRUD class
task = CRUDTask(Task)
```

## Creating API Endpoints

Next, create API endpoints to expose your model. Follow these steps:

### Step 1: Create a Route File

Create a new file `app/api/routes/tasks.py`:

```python
from fastapi import APIRouter, HTTPException, status
from sqlmodel import Session
from uuid import UUID
from typing import Any

from app import crud, models
from app.api import deps

router = APIRouter()


@router.get("/", response_model=models.TasksPublic)
def read_tasks(
    db: deps.SessionDep,
    current_user: deps.CurrentUser,
    skip: int = 0,
    limit: int = 100,
) -> Any:
    """
    Retrieve tasks for the current user.
    """
    tasks = crud.task.get_multi_by_owner(
        db, owner_id=current_user.id, skip=skip, limit=limit
    )
    count = len(tasks)
    return models.TasksPublic(data=tasks, count=count)


@router.post("/", response_model=models.TaskPublic)
def create_task(
    *,
    db: deps.SessionDep,
    current_user: deps.CurrentUser,
    task_in: models.TaskCreate,
) -> Any:
    """
    Create new task for the current user.
    """
    task = crud.task.create_with_owner(
        db=db, obj_in=task_in, owner_id=current_user.id
    )
    return task


@router.get("/{task_id}", response_model=models.TaskPublic)
def read_task(
    *,
    db: deps.SessionDep,
    current_user: deps.CurrentUser,
    task_id: UUID,
) -> Any:
    """
    Get a specific task by id.
    """
    task = crud.task.get(db=db, id=task_id)
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, detail="Task not found"
        )
    if task.owner_id != current_user.id:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN, detail="Not enough permissions"
        )
    return task


@router.put("/{task_id}", response_model=models.TaskPublic)
def update_task(
    *,
    db: deps.SessionDep,
    current_user: deps.CurrentUser,
    task_id: UUID,
    task_in: models.TaskUpdate,
) -> Any:
    """
    Update a task.
    """
    task = crud.task.get(db=db, id=task_id)
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, detail="Task not found"
        )
    if task.owner_id != current_user.id:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN, detail="Not enough permissions"
        )
    task = crud.task.update(db=db, db_obj=task, obj_in=task_in)
    return task


@router.delete("/{task_id}", response_model=models.Message)
def delete_task(
    *,
    db: deps.SessionDep,
    current_user: deps.CurrentUser,
    task_id: UUID,
) -> Any:
    """
    Delete a task.
    """
    task = crud.task.get(db=db, id=task_id)
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, detail="Task not found"
        )
    if task.owner_id != current_user.id:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN, detail="Not enough permissions"
        )
    task = crud.task.remove(db=db, id=task_id)
    return {"message": "Task deleted successfully"}
```

### Step 2: Register the Router

Add your router to `app/api/main.py`:

```python
from fastapi import APIRouter

from app.api.routes import items, login, users, utils, private, tasks  # Add tasks here

api_router = APIRouter()
api_router.include_router(login.router, prefix="/login", tags=["login"])
api_router.include_router(users.router, prefix="/users", tags=["users"])
api_router.include_router(utils.router, prefix="/utils", tags=["utils"])
api_router.include_router(private.router, prefix="/private", tags=["private"])
api_router.include_router(items.router, prefix="/items", tags=["items"])
api_router.include_router(tasks.router, prefix="/tasks", tags=["tasks"])  # Add this line
```

## Authentication and Permissions

### Using Authentication

The application uses JWT-based authentication. To protect routes:

1. Always use the built-in dependency injection system for authentication:

```python
from app.api import deps

@router.get("/some-protected-route")
def protected_endpoint(
    current_user: deps.CurrentUser,  # This injects the authenticated user
):
    # Your code here
    return {"user_id": current_user.id}
```

2. For superuser-only endpoints:

```python
@router.get("/admin-only")
def admin_endpoint(
    current_user: Annotated[
        models.User, Depends(deps.get_current_active_superuser)
    ],
):
    # Only superusers can access this
    return {"message": "Hello, admin!"}
```

### Custom Permission Logic

For more complex permissions:

```python
def check_task_owner(
    task_id: UUID,
    current_user: models.User,
    db: Session,
) -> models.Task:
    """Check if current user is the task owner."""
    task = crud.task.get(db=db, id=task_id)
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    if task.owner_id != current_user.id:
        raise HTTPException(status_code=403, detail="Not enough permissions")
    return task

# Then in your endpoint
@router.get("/{task_id}")
def read_task(
    task_id: UUID,
    current_user: deps.CurrentUser,
    db: deps.SessionDep,
):
    task = check_task_owner(task_id, current_user, db)
    return task
```

## Working with Database Relationships

The application uses SQLModel for defining relationships between tables. Here's how to work with them:

### One-to-Many Relationships

As shown in our Task example, we defined:

1. Relationship in the parent model (`User`):
   ```python
   tasks: list["Task"] = Relationship(back_populates="owner")
   ```

2. Relationship in the child model (`Task`):
   ```python
   owner_id: uuid.UUID = Field(foreign_key="user.id")
   owner: User | None = Relationship(back_populates="tasks")
   ```

3. Using relationships in queries:
   ```python
   # Get all tasks for a user
   user = db.get(User, user_id)
   tasks = user.tasks  # Direct access via relationship
   
   # Get the owner of a task
   task = db.get(Task, task_id)
   owner = task.owner
   ```

### Many-to-Many Relationships

For many-to-many relationships, create a link table:

```python
class TaskTag(SQLModel, table=True):
    task_id: uuid.UUID = Field(
        foreign_key="task.id", primary_key=True
    )
    tag_id: uuid.UUID = Field(
        foreign_key="tag.id", primary_key=True
    )

class Tag(SQLModel, table=True):
    id: uuid.UUID = Field(default_factory=uuid.uuid4, primary_key=True)
    name: str = Field(index=True)
    tasks: list["Task"] = Relationship(
        back_populates="tags",
        link_model=TaskTag,
    )

# Update Task model
class Task(TaskBase, table=True):
    # ... existing fields
    tags: list["Tag"] = Relationship(
        back_populates="tasks",
        link_model=TaskTag,
    )
```

## Error Handling

Consistent error handling is crucial for a good API. Follow these patterns:

### HTTP Exceptions

Use FastAPI's HTTPException for API errors:

```python
from fastapi import HTTPException, status

# Not found error
if not item:
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="Item not found"
    )

# Permission error
if item.owner_id != current_user.id:
    raise HTTPException(
        status_code=status.HTTP_403_FORBIDDEN,
        detail="Not enough permissions"
    )

# Validation error
if user_exists:
    raise HTTPException(
        status_code=status.HTTP_400_BAD_REQUEST,
        detail="Email already registered"
    )
```

### Exception Handlers

For custom exception handling, add handlers in `main.py`:

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

class CustomException(Exception):
    def __init__(self, name: str):
        self.name = name

app = FastAPI()

@app.exception_handler(CustomException)
async def custom_exception_handler(request: Request, exc: CustomException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something wrong."},
    )
```

## Email Integration

To send emails for features like password reset:

1. Ensure SMTP settings are configured in `.env` file:
   ```
   SMTP_TLS=True
   SMTP_PORT=587
   SMTP_HOST=smtp.example.com
   SMTP_USER=your-email@example.com
   SMTP_PASSWORD=your-password
   EMAILS_FROM_EMAIL=your-email@example.com
   ```

2. Use the built-in email utilities:
   ```python
   from app.utils import send_email
   
   send_email(
       email_to=user.email,
       subject="Password Reset",
       template_name="password_reset",
       template_vars={
           "project_name": settings.PROJECT_NAME,
           "username": user.email,
           "reset_token": token,
           "valid_hours": settings.EMAIL_RESET_TOKEN_EXPIRE_HOURS,
       },
   )
   ```

3. Create email templates in `app/email-templates/src/` directory using MJML.

## Testing Your Code

Follow these steps to test your new features:

### Unit Tests

1. Create test files in `app/tests/` directory:
   ```python
   # app/tests/api/test_tasks.py
   from fastapi.testclient import TestClient
   from sqlmodel import Session
   
   from app.core.config import settings
   from app.tests.utils.user import create_random_user
   from app.tests.utils.task import create_random_task
   
   def test_create_task(
       client: TestClient, superuser_token_headers: dict[str, str], db: Session
   ) -> None:
       data = {"title": "Test Task", "description": "Test Description"}
       response = client.post(
           f"{settings.API_V1_STR}/tasks/", headers=superuser_token_headers, json=data,
       )
       assert response.status_code == 200
       content = response.json()
       assert content["title"] == data["title"]
       assert content["description"] == data["description"]
       assert "id" in content
       assert "owner_id" in content
   ```

2. Create test utilities in `app/tests/utils/`:
   ```python
   # app/tests/utils/task.py
   import random
   import string
   from typing import Optional
   
   from sqlmodel import Session
   
   from app import crud, models
   from app.tests.utils.user import create_random_user
   
   def create_random_task(
       db: Session, *, owner_id: Optional[str] = None
   ) -> models.Task:
       if owner_id is None:
           user = create_random_user(db)
           owner_id = user.id
       title = "".join(random.choices(string.ascii_lowercase, k=10))
       description = "".join(random.choices(string.ascii_lowercase, k=30))
       task_in = models.TaskCreate(title=title, description=description)
       return crud.task.create_with_owner(db=db, obj_in=task_in, owner_id=owner_id)
   ```

3. Run tests:
   ```bash
   # Inside the backend container
   pytest app/tests/api/test_tasks.py -v
   ```

### Integration Tests

Integration tests ensure your API works end-to-end:

```python
def test_task_workflow(
    client: TestClient, normal_user_token_headers: dict[str, str], db: Session
) -> None:
    # Create a task
    task_data = {"title": "Integration Test", "description": "Testing the full flow"}
    response = client.post(
        f"{settings.API_V1_STR}/tasks/", 
        headers=normal_user_token_headers, 
        json=task_data,
    )
    assert response.status_code == 200
    task = response.json()
    task_id = task["id"]
    
    # Retrieve the task
    response = client.get(
        f"{settings.API_V1_STR}/tasks/{task_id}", 
        headers=normal_user_token_headers,
    )
    assert response.status_code == 200
    
    # Update the task
    update_data = {"title": "Updated Task"}
    response = client.put(
        f"{settings.API_V1_STR}/tasks/{task_id}", 
        headers=normal_user_token_headers, 
        json=update_data,
    )
    assert response.status_code == 200
    assert response.json()["title"] == update_data["title"]
    
    # Delete the task
    response = client.delete(
        f"{settings.API_V1_STR}/tasks/{task_id}", 
        headers=normal_user_token_headers,
    )
    assert response.status_code == 200
    
    # Verify it's deleted
    response = client.get(
        f"{settings.API_V1_STR}/tasks/{task_id}", 
        headers=normal_user_token_headers,
    )
    assert response.status_code == 404
```

## Frontend Integration

This section explains how to expose your backend API to the frontend effectively.

### API Documentation

FastAPI automatically generates OpenAPI documentation. Access it at:

```
http://localhost:8000/api/v1/docs
```

This interactive documentation helps frontend developers understand your API.

### CORS Configuration

CORS is already configured in `app/main.py` and `app/core/config.py`. Ensure your frontend origin is included in the `BACKEND_CORS_ORIGINS` environment variable.

### Authentication Flow

The frontend should:

1. Call `/api/v1/login/access-token` with user credentials to get a token
2. Include the token in all subsequent requests as a Bearer token in the Authorization header:
   ```
   Authorization: Bearer <token>
   ```

### Implementing Frontend API Calls

Example using fetch API in JavaScript:

```javascript
// Login
async function login(email, password) {
  const response = await fetch('/api/v1/login/access-token', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: new URLSearchParams({
      username: email,
      password: password,
    }),
  });
  return await response.json();
}

// Get tasks with authentication
async function getTasks(token) {
  const response = await fetch('/api/v1/tasks/', {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });
  return await response.json();
}

// Create a new task
async function createTask(token, taskData) {
  const response = await fetch('/api/v1/tasks/', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(taskData),
  });
  return await response.json();
}
```

### TypeScript Interfaces

For TypeScript frontends, create interfaces based on your models:

```typescript
// src/types/api.ts

export interface Task {
  id: string;
  title: string;
  description: string | null;
  is_completed: boolean;
  due_date: string | null;
  priority: number;
  created_at: string;
  updated_at: string | null;
  owner_id: string;
}

export interface TasksResponse {
  data: Task[];
  count: number;
}

export interface CreateTaskPayload {
  title: string;
  description?: string;
  due_date?: string;
  priority?: number;
}

export interface UpdateTaskPayload {
  title?: string;
  description?: string;
  is_completed?: boolean;
  due_date?: string;
  priority?: number;
}
```

## Troubleshooting

### Common Issues and Solutions

#### Database Migrations

**Issue**: Migration generates unwanted changes or errors

**Solution**:
1. Delete the migration file if it wasn't applied yet
2. Check for inconsistencies between models and database
3. Run `alembic revision --autogenerate --verbose` to see detailed output

#### Authentication Issues

**Issue**: Token validation fails

**Solution**:
1. Check `SECRET_KEY` in environment variables
2. Verify token expiration in settings
3. Ensure correct JWT algorithm is used

#### CORS Issues

**Issue**: Frontend receives CORS errors

**Solution**:
1. Add frontend origin to `BACKEND_CORS_ORIGINS` in `.env`
2. Check if the frontend is using the exact same origin as configured
3. Verify that credentials mode is correctly set in fetch requests

#### Database Connection Issues

**Issue**: Cannot connect to database

**Solution**:
1. Verify database credentials in `.env`
2. Check if database service is running
3. Ensure network connectivity between services

### Debugging Tips

1. Use FastAPI's built-in debug mode
2. Add print statements or use a logger
3. Check logs with `docker compose logs backend`
4. For database issues, use direct SQL queries to validate data
5. Use VS Code's debugging features with the provided launch configurations
