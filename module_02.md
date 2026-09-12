# Module 2: Enterprise Backend with FastAPI 

**Duration:** 4 Hours (1.5 hrs Lecture/Live Code + 2.5 hrs Lab)  
**Domain Context:** HR Payroll Calculation Engine  
**Objective:** Move from basic Python to building robust, typed REST APIs using FastAPI, comparing the architecture directly to ASP.NET Core.

---

## 📖 Part 1: Lecture Notes (Concepts)

### 1. Why FastAPI?
You already know how to build APIs in C# ASP.NET Core. We need a Python equivalent that is just as robust to serve as our AI Sidecar. We are choosing **FastAPI** over Flask because:
*   It is incredibly fast (built on asynchronous Python).
*   It automatically generates a Swagger UI page (just like ASP.NET Core).
*   It forces you to use strict data types.

### 2. C# to Python Translation Guide
Let's map what you know in C# to FastAPI:

| Concept | C# (ASP.NET Core) | Python (FastAPI) |
| :--- | :--- | :--- |
| **API Framework** | `Microsoft.AspNetCore.Mvc` | `fastapi` |
| **Data Models (DTOs)** | `public class Model { get; set; }` | `pydantic.BaseModel` |
| **Routing / Endpoints**| `[HttpGet("/api/users")]` | `@app.get("/api/users")` |
| **Running the Server** | `dotnet run` | `uvicorn main:app --reload` |

### 3. What is Pydantic?
In standard Python, variables can be anything (a string can suddenly become an integer). In Enterprise APIs, this causes crashes. **Pydantic** is a library built into FastAPI that enforces strict data types, acting exactly like C# DTOs (Data Transfer Objects).

---

## 💻 Part 2: Live Code-Along (Follow the Mentor)
> **Mentor Note:** Ensure students install the requirements first: `pip install fastapi uvicorn pydantic`. Type this live and show them the magic of the auto-generated Swagger page.

**File:** `live_api.py`
```python
from fastapi import FastAPI
from pydantic import BaseModel

# 1. Initialize the API app
app = FastAPI(title="HR Internal API", version="1.0")

# 2. Define a Pydantic Model (Like a C# DTO)
class EmployeeInfo(BaseModel):
    name: str
    department: str
    age: int

# 3. Create a GET Endpoint
@app.get("/")
def health_check():
    return {"status": "API is running successfully!"}

# 4. Create a POST Endpoint
@app.post("/api/employees")
def register_employee(employee: EmployeeInfo):
    # Because of Pydantic, we get auto-complete and type safety here!
    message = f"Registered {employee.name} to the {employee.department} department."
    
    return {
        "success": True,
        "message": message,
        "employee_age": employee.age
    }
```
**How to run it:** Open your terminal and type:
```bash
uvicorn live_api:app --reload
```
**The Magic Trick:** Tell students to open their browsers and go to `http://127.0.0.1:8000/docs`. They will see a fully interactive **Swagger UI** where they can test their POST request without needing Postman yet!

---

## 🧪 Part 3: Student Lab Activity (2.5 Hours)

**Scenario:** 
You are building the backend calculation engine for the **HR Payroll System**. The frontend team will send you an employee's base salary and their overtime (OT) hours. Your API must calculate their gross pay, deduct taxes, and return the final net pay.

**Your Tasks:**
1.  **Setup:** Create a new file called `payroll_api.py` and initialize a FastAPI app.
2.  **Create the Request Model (DTO):** 
    *   Create a Pydantic model named `PayrollRequest`.
    *   It should expect: `employee_id` (string), `base_salary` (float), and `ot_hours` (int).
3.  **Create the Response Model (DTO):**
    *   Create a Pydantic model named `PayrollResponse`.
    *   It should return: `employee_id` (string), `gross_pay` (float), `tax_deduction` (float), and `net_pay` (float).
4.  **Build the Calculation Endpoint:**
    *   Create a `@app.post("/api/payroll/calculate")` route.
    *   **The Math Logic:** 
        *   OT Rate is $20 per OT hour.
        *   `gross_pay` = `base_salary` + (`ot_hours` * 20).
        *   `tax_deduction` = 10% (0.10) of the `gross_pay`.
        *   `net_pay` = `gross_pay` - `tax_deduction`.
    *   Return the `PayrollResponse` model.
5.  **Test:** Run the server using Uvicorn, go to `/docs`, and test your math!

---

## 🔐 Part 4: Mentor's Answer Key (Solution)
> **Mentor Note:** Walk around and make sure students are defining the return type in their function signature `-> PayrollResponse:` as it helps FastAPI document the Swagger UI perfectly.

**File:** `lab_solution.py`
```python
from fastapi import FastAPI
from pydantic import BaseModel

# Initialize App
app = FastAPI(title="Payroll Calculation Engine")

# --- 1. Request & Response Models (DTOs) ---
class PayrollRequest(BaseModel):
    employee_id: str
    base_salary: float
    ot_hours: int

class PayrollResponse(BaseModel):
    employee_id: str
    gross_pay: float
    tax_deduction: float
    net_pay: float

# --- 2. Calculation Endpoint ---
# Notice how we specify the response_model in the decorator for perfect Swagger documentation
@app.post("/api/payroll/calculate", response_model=PayrollResponse)
def calculate_payroll(request: PayrollRequest):
    
    # 1. Calculate Overtime Pay ($20/hr)
    ot_pay = request.ot_hours * 20.0
    
    # 2. Calculate Gross Pay
    gross_pay = request.base_salary + ot_pay
    
    # 3. Calculate Tax (10% flat rate)
    tax_deduction = gross_pay * 0.10
    
    # 4. Calculate Net Pay
    net_pay = gross_pay - tax_deduction
    
    # 5. Return the Response Model
    return PayrollResponse(
        employee_id=request.employee_id,
        gross_pay=gross_pay,
        tax_deduction=tax_deduction,
        net_pay=net_pay
    )
```

***

### End of Module 2
**Mentor Check-in:** Remind students how powerful this is. We just built an enterprise-ready, strongly-typed API with auto-generated documentation in less than 40 lines of code. 
*Next up in Module 3: We will learn how to write automated tests for this exact API using `pytest`, Postman, and Selenium!*

---
*(Let me know when you are ready to generate **Module 3: Quality Assurance & UI Automation**!)*
