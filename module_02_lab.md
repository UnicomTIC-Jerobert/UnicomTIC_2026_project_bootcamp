# 🧪 PART 2: LAB ACTIVITY (3 Hours)

**Scenario:** 
You are tasked with building the secure backend calculation engine for the **HR Payroll System**. The frontend team needs full CRUD endpoints to manage payroll. The API must calculate taxes automatically, store records in a SQLite database, and strictly protect modification endpoints using JWT Authentication.

### Phase 1: Setup & Database Models
1. Create a new Python file: `payroll_api.py`.
2. Install requirements: `pip install fastapi uvicorn pydantic sqlalchemy pyjwt`
3. Setup SQLAlchemy to connect to `sqlite:///./payroll.db`.
4. Create an Entity Model (`PayrollDB`) with columns: `id` (Integer), `employee_name` (String), `base_salary` (Float), `ot_hours` (Integer), and `net_pay` (Float).

### Phase 2: Pydantic DTOs
1. Create `PayrollCreate` (Accepts name, base salary, and OT hours).
2. Create `PayrollUpdate` (Accepts base salary and OT hours for updating).
3. Create `PayrollResponse` (Returns all fields, including the calculated net pay and ID. Remember to add `orm_mode = True`).

### Phase 3: JWT Authentication System
1. Create a hardcoded secret key (e.g., `"hr_super_secret"`).
2. Create a `POST /login` endpoint that accepts a `username` and returns a signed JWT token.
3. Create a dependency function `verify_token(token = Depends(...))` that decodes the token and acts as your `[Authorize]` guard.

### Phase 4: Full CRUD Endpoints
Implement the following REST endpoints. **Important Math Logic:** Net Pay = Base Salary + (OT Hours * 20) - 10% Tax.

1.  **Create (POST `/payroll`)** -> *[PROTECTED BY JWT]*: Takes the payload, runs the math, saves to DB, returns the saved record.
2.  **Read All (GET `/payroll`)**: Returns a list of all records.
3.  **Read One (GET `/payroll/{id}`)**: Returns a single record or a `404 Not Found`.
4.  **Update (PUT `/payroll/{id}`)** -> *[PROTECTED BY JWT]*: Finds the record, recalculates the Net Pay with the new data, saves, and returns the updated record.
5.  **Delete (DELETE `/payroll/{id}`)** -> *[PROTECTED BY JWT]*: Removes the record from the DB.

### Phase 5: Testing
Run your app (`uvicorn payroll_api:app --reload`). Open Swagger UI (`/docs`). Prove that you cannot Create, Update, or Delete without first generating a token via the `/login` endpoint and pasting it into the Swagger "Authorize" padlock!

***
***

# 🔐 PART 3: MENTOR'S COMPLETE SOLUTION CODE
> **Mentor Note:** Keep this solution hidden until the end of the lab. Walk through the code line-by-line to show how neatly FastAPI handles ORM, Validation, and Auth in a single file.

**File:** `payroll_api_solution.py`

```python
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel
from sqlalchemy import create_engine, Column, Integer, String, Float
from sqlalchemy.orm import declarative_base, sessionmaker, Session
import jwt

# ==========================================
# 1. DATABASE & ORM SETUP (SQLAlchemy)
# ==========================================
SQLALCHEMY_DATABASE_URL = "sqlite:///./payroll.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Entity Model (Maps to SQL Table)
class PayrollDB(Base):
    __tablename__ = "payroll_records"
    id = Column(Integer, primary_key=True, index=True)
    employee_name = Column(String, index=True)
    base_salary = Column(Float)
    ot_hours = Column(Integer)
    net_pay = Column(Float)

Base.metadata.create_all(bind=engine)

# DB Dependency (Injects DB Session into endpoints)
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# ==========================================
# 2. DATA VALIDATION (Pydantic DTOs)
# ==========================================
class PayrollCreate(BaseModel):
    employee_name: str
    base_salary: float
    ot_hours: int

class PayrollUpdate(BaseModel):
    base_salary: float
    ot_hours: int

class PayrollResponse(BaseModel):
    id: int
    employee_name: str
    base_salary: float
    ot_hours: int
    net_pay: float

    class Config:
        orm_mode = True # Crucial: tells Pydantic to read SQLAlchemy objects

# ==========================================
# 3. JWT AUTHENTICATION SYSTEM
# ==========================================
SECRET_KEY = "hr_super_secret_key"
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="login")

app = FastAPI(title="HR Payroll System", version="1.0")

# Login Endpoint to generate token
@app.post("/login", tags=["Authentication"])
def login(username: str):
    token = jwt.encode({"sub": username}, SECRET_KEY, algorithm="HS256")
    return {"access_token": token, "token_type": "bearer"}

# Auth Dependency (Acts like [Authorize])
def verify_token(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload["sub"]
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token has expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")

# ==========================================
# 4. BUSINESS LOGIC HELPER
# ==========================================
def calculate_net_pay(base_salary: float, ot_hours: int) -> float:
    gross_pay = base_salary + (ot_hours * 20.0)
    tax = gross_pay * 0.10
    return gross_pay - tax

# ==========================================
# 5. REST CRUD ENDPOINTS
# ==========================================

# CREATE (POST) - Protected
@app.post("/payroll", response_model=PayrollResponse, tags=["Payroll CRUD"])
def create_payroll(request: PayrollCreate, db: Session = Depends(get_db), current_user: str = Depends(verify_token)):
    net_pay = calculate_net_pay(request.base_salary, request.ot_hours)
    
    new_record = PayrollDB(
        employee_name=request.employee_name,
        base_salary=request.base_salary,
        ot_hours=request.ot_hours,
        net_pay=net_pay
    )
    db.add(new_record)
    db.commit()
    db.refresh(new_record)
    return new_record

# READ ALL (GET) - Open Access
@app.get("/payroll", response_model=list[PayrollResponse], tags=["Payroll CRUD"])
def get_all_payroll(db: Session = Depends(get_db)):
    return db.query(PayrollDB).all()

# READ ONE (GET) - Open Access
@app.get("/payroll/{id}", response_model=PayrollResponse, tags=["Payroll CRUD"])
def get_payroll_by_id(id: int, db: Session = Depends(get_db)):
    record = db.query(PayrollDB).filter(PayrollDB.id == id).first()
    if not record:
        raise HTTPException(status_code=404, detail="Payroll record not found")
    return record

# UPDATE (PUT) - Protected
@app.put("/payroll/{id}", response_model=PayrollResponse, tags=["Payroll CRUD"])
def update_payroll(id: int, request: PayrollUpdate, db: Session = Depends(get_db), current_user: str = Depends(verify_token)):
    record = db.query(PayrollDB).filter(PayrollDB.id == id).first()
    if not record:
        raise HTTPException(status_code=404, detail="Payroll record not found")
    
    # Update fields and recalculate
    record.base_salary = request.base_salary
    record.ot_hours = request.ot_hours
    record.net_pay = calculate_net_pay(request.base_salary, request.ot_hours)
    
    db.commit()
    db.refresh(record)
    return record

# DELETE (DELETE) - Protected
@app.delete("/payroll/{id}", tags=["Payroll CRUD"])
def delete_payroll(id: int, db: Session = Depends(get_db), current_user: str = Depends(verify_token)):
    record = db.query(PayrollDB).filter(PayrollDB.id == id).first()
    if not record:
        raise HTTPException(status_code=404, detail="Payroll record not found")
    
    db.delete(record)
    db.commit()
    return {"message": f"Payroll record {id} successfully deleted"}
```
