# Module 2: Enterprise Backend with FastAPI & SQLite

**Duration:** 5 Hours (2 hrs Lecture + 3 hrs Lab Activity)  
**Domain Context:** Secure HR Payroll CRUD Engine  
**Objective:** Master modern Python API development by building a fully functional, database-backed REST API with JWT authentication, leveraging your existing ASP.NET Core knowledge.

---

# 📖 PART 1: LECTURE MATERIAL (2 Hours)

## Topic 1: The Great Framework Comparison
Before we code, we need to understand *why* we are using FastAPI to build our AI sidecar, rather than sticking to C# or using older Python frameworks.

### 1. C# ASP.NET Core (The Enterprise Monolith)
*   **Pros:** Highly structured, built-in Dependency Injection, strict typing, Entity Framework Core is powerful, incredibly secure.
*   **Cons:** Heavy boilerplate. Setting up a simple microservice takes a lot of files and configuration. Not natively built for the Python AI ecosystem (LangChain, HuggingFace, etc.).

### 2. Python Flask (The Micro-Framework)
*   **Pros:** Very easy to learn. A basic API can be written in 5 lines of code.
*   **Cons:** **No strict typing.** If a user sends `"age": "twenty"`, Flask accepts it, and your app crashes during calculation. Generating Swagger UI requires third-party plugins. It uses older synchronous logic (WSGI).

### 3. Python FastAPI (The Modern Standard)
FastAPI is the perfect bridge for C# developers moving to Python.
*   **Strict Typing:** Uses `Pydantic` to enforce data types, exactly like C# DTOs.
*   **Auto-Documentation:** Automatically generates interactive Swagger UI (`/docs`).
*   **Asynchronous:** Built on ASGI, making it as fast as Node.js or Go.
*   **Dependency Injection:** Has a built-in DI system (using the `Depends()` keyword) that acts very much like ASP.NET Core's service container.

---

## Topic 2: RESTful Methods (CRUD Mapping)
Just like in C# Web API, we map HTTP verbs to database actions. FastAPI uses simple decorators for this:
*   **POST (`@app.post`)** -> **Create:** Add a new employee to the database.
*   **GET (`@app.get`)** -> **Read:** Fetch employee(s) from the database.
*   **PUT (`@app.put`)** -> **Update:** Completely replace an existing employee record.
*   **DELETE (`@app.delete`)** -> **Delete:** Remove the record.

---

## Topic 3: Deep Dive into Pydantic (Data Validation)
In C#, you use `public class EmployeeDTO` with Data Annotations like `[Required]` or `[Range]`. 
In FastAPI, we use **Pydantic**. 

Pydantic automatically validates incoming JSON payloads. If the payload is wrong, FastAPI automatically returns a `422 Unprocessable Entity` error. You don't have to write `if(request.Age == null)` anymore!

**C# Concept:**
```csharp
public class PayrollRequest {
    [Required]
    public string EmployeeName { get; set; }
    public float BaseSalary { get; set; }
}
```
**FastAPI Equivalent (Pydantic):**
```python
from pydantic import BaseModel, Field

class PayrollRequest(BaseModel):
    employee_name: str
    base_salary: float = Field(gt=0, description="Salary must be greater than zero")
```

---

## Topic 4: Database Storage (SQLite & SQLAlchemy)
We don't want to install SQL Server for these Python sidecars. We will use **SQLite** (a lightweight, file-based database). 
To interact with the database, we use **SQLAlchemy**, which is Python's equivalent of **Entity Framework Core**.

*   **`engine`:** The connection to the database file.
*   **`Session`:** Equivalent to the C# `DbContext`. It tracks changes and commits them.
*   **`Base`:** Equivalent to a C# Entity Class mapping to a SQL table.

---

## Topic 5: JWT Authentication in FastAPI
In C#, to protect an endpoint, you add the `[Authorize]` attribute above your controller.
In FastAPI, we use **Dependency Injection** via the `Depends()` keyword to achieve the exact same thing.

If a user hits a protected endpoint without a valid JWT token, the Dependency intercepts the request and throws a `401 Unauthorized` before the function even runs.

*(We will implement this fully in the Lab Activity).*

***
***
