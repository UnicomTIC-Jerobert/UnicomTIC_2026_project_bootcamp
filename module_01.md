Here is the complete **Mentor's Teaching Kit for Module 1**. 

It is formatted in clean, structured Markdown. You can copy this entire block, save it as `Module-1-OOP-and-Streamlit.md`, push it to your GitHub repository, and project it directly onto the screen as your teaching material!

***

# Module 1: Python OOP & Rapid UI Development with Streamlit

**Duration:** 4 Hours (1.5 hrs Lecture/Live Code + 2.5 hrs Lab)  
**Domain Context:** HR Leave Management System  
**Objective:** Refresh Object-Oriented Python and learn to build instant UIs without needing Angular/HTML.

---

## 📖 Part 1: Lecture Notes (Concepts)

### 1. Python OOP vs. C# OOP
You already know C#. Let's translate that knowledge into Python.
*   **Classes:** Python doesn't use `public` or `private` keywords or curly braces `{}`. It uses indentation.
*   **Constructors:** In C#, the constructor is the class name. In Python, it is *always* `__init__`.
*   **The `this` keyword:** In C#, you use `this.Name`. In Python, you must explicitly pass `self` into every method, and use `self.name`.

**C# Example:**
```csharp
public class Employee {
    public string Name;
    public Employee(string name) {
        this.Name = name;
    }
}
```
**Python Equivalent:**
```python
class Employee:
    def __init__(self, name):
        self.name = name
```

### 2. What is Streamlit?
When building AI applications, waiting for the frontend team to build an Angular UI takes too long. We need to test our AI *now*.
*   **Streamlit** is a Python library that turns Python scripts into interactive web apps in minutes.
*   **No HTML, No CSS, No JavaScript required.** 
*   It reads your code top-to-bottom. If you write `st.write("Hello")`, it puts text on the webpage. If you write `st.button("Click")`, it draws a button.

---

## 💻 Part 2: Live Code-Along (Follow the Mentor)
> **Mentor Note:** Open VS Code, create a virtual environment, install streamlit (`pip install streamlit`), and type this live with the students.

**File:** `live_demo.py`
```python
import streamlit as st

# 1. Define our Python Class
class Employee:
    def __init__(self, name, role):
        self.name = name
        self.role = role

    def get_details(self):
        return f"{self.name} works as a {self.role}."

# 2. Build the Streamlit UI
st.title("👨‍💼 HR Employee Directory Prototype")
st.write("Welcome to the internal HR portal.")

# 3. Create a simple input form
employee_name = st.text_input("Enter Employee Name:")
employee_role = st.selectbox("Select Role:", ["Software Engineer", "HR Manager", "QA Tester"])

# 4. Handle Button Click
if st.button("Register Employee"):
    # Instantiate the Object
    new_employee = Employee(name=employee_name, role=employee_role)
    
    # Display the output on the UI
    st.success("Employee Registered Successfully!")
    st.info(new_employee.get_details())
```
**How to run it:** Open your terminal and type:
```bash
streamlit run live_demo.py
```
*(A browser window will automatically pop up with your working web app!)*

---

## 🧪 Part 3: Student Lab Activity (2.5 Hours)

**Scenario:** 
You are tasked with building the frontend prototype for the **HR Leave Management System**. Employees need a web page to submit their leave requests.

**Your Tasks:**
1.  **Create a Python Class:** Create a class called `LeaveRequest`.
    *   It should have an `__init__` method that accepts: `employee_name`, `leave_type`, and `duration_days`.
    *   Create a method inside the class called `get_summary()` that returns a string (e.g., *"John Doe requested 3 days of Sick Leave."*).
2.  **Build the Streamlit UI:**
    *   Add a title and a description to the page.
    *   Create an input form for the user to type their Name.
    *   Create a dropdown (`st.selectbox`) for Leave Type ("Annual", "Sick", "Casual").
    *   Create a number input (`st.number_input`) for the number of days.
3.  **The Action:**
    *   Add a "Submit Leave Request" button.
    *   When clicked, instantiate your `LeaveRequest` class using the inputs.
    *   Display a warning card (`st.warning()`) that says "Status: Pending Manager Approval", followed by the summary from your class method.

---

## 🔐 Part 4: Mentor's Answer Key (Solution)
> **Mentor Note:** Keep this hidden until the end of the lab, then review it together. Point out the use of `st.form`, which is a best practice in Streamlit to group inputs together.

**File:** `lab_solution.py`
```python
import streamlit as st

# --- 1. OOP Logic ---
class LeaveRequest:
    def __init__(self, employee_name, leave_type, duration_days):
        self.employee_name = employee_name
        self.leave_type = leave_type
        self.duration_days = duration_days

    def get_summary(self):
        return f"{self.employee_name} has requested {self.duration_days} day(s) of {self.leave_type} leave."

# --- 2. Streamlit UI ---
st.set_page_config(page_title="Leave Portal", page_icon="📅")
st.title("📅 Employee Leave Request Portal")
st.write("Please fill out the form below to apply for leave.")

# Using st.form is best practice so the page doesn't refresh on every single keystroke
with st.form("leave_form"):
    st.subheader("Leave Details")
    
    # UI Inputs
    emp_name = st.text_input("Full Name:")
    l_type = st.selectbox("Leave Type:", ["Annual Leave", "Sick Leave", "Casual Leave"])
    d_days = st.number_input("Number of Days:", min_value=1, max_value=30, value=1)
    
    # Form Submit Button
    submitted = st.form_submit_button("Submit Request")

# --- 3. Action Handling ---
if submitted:
    if emp_name == "":
        st.error("Please enter your name!")
    else:
        # Instantiate the class
        request_obj = LeaveRequest(employee_name=emp_name, leave_type=l_type, duration_days=d_days)
        
        # Display Results
        st.warning("⏳ Status: Pending Manager Approval")
        st.info(request_obj.get_summary())
        st.success("Your request has been logged in the system.")
```

***

### End of Module 1
**Mentor Check-in:** Ask the students how building a UI in Python feels compared to Angular. 
*Next up in Module 2: We will build the actual REST API backend for our HR Payroll System using FastAPI!*

---
*(Let me know when you are ready to generate **Module 2: Enterprise Backend with FastAPI** in this exact same format!)*
