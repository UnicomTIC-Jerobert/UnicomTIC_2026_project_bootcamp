# Module 1: Python Fundamentals, Deep OOP & Streamlit UI Mastery

**Total Duration:** 4 Hours  
**Repository Path:** `/docs/module-1-python-streamlit.md`

---

# 📘 Lecture 1: Python Basics & OOP vs. C# (2 Hours)

Since you are coming from a strongly-typed C# and Angular background, Python will feel very different. Python is dynamically typed, concise, and relies on indentation rather than curly braces `{}`.

## 1. Python Basics Refresher

### Variables & Data Types
In C#, you declare types (`int`, `string`). In Python, the type is inferred at runtime.
```python
# Python
age = 25                  # int
name = "John Doe"         # str
is_active = True          # bool (Note: Capital 'T')
```

### Data Structures (Lists vs. Arrays, Dictionaries vs. HashMaps)
```python
# Lists (Like C# List<T> or Array)
departments = ["HR", "IT", "Finance"]
departments.append("Sales")

# Dictionaries (Like C# Dictionary<string, string>)
employee = {
    "name": "Sarah",
    "role": "QA Engineer",
    "salary": 75000
}
print(employee["role"]) # Output: QA Engineer
```

### Control Flow & Functions
Notice the lack of curly braces `{}`. Indentation (tabs/spaces) defines the code block.
```python
# Functions use the 'def' keyword
def calculate_bonus(salary, performance_rating):
    if performance_rating > 4.5:
        return salary * 0.20
    elif performance_rating > 3.0:
        return salary * 0.10
    else:
        return 0

bonus = calculate_bonus(75000, 4.8)
```

---

## 2. Deep Dive: Object-Oriented Programming (C# vs. Python)

### A. Classes and Constructors
In C#, the constructor is the same name as the class. In Python, it is a special "dunder" (double underscore) method called `__init__`. You **must** pass `self` (equivalent to `this` in C#) as the first parameter to every instance method.

| Feature | C# | Python |
| :--- | :--- | :--- |
| **Constructor** | `public Employee()` | `def __init__(self):` |
| **Current Instance**| `this.Name` | `self.name` |
| **Instantiation** | `new Employee()` | `Employee()` (No 'new' keyword) |

```python
class Employee:
    # Constructor
    def __init__(self, name, department):
        self.name = name
        self.department = department

    # Instance Method
    def get_info(self):
        return f"{self.name} works in {self.department}"

# Creating an object
emp1 = Employee("John", "IT")
```

### B. Access Modifiers (Public, Private, Protected)
C# enforces access strictly using `public`, `private`, and `protected`. Python operates on a "gentleman's agreement" using underscores.

*   **Public:** `self.name` (Accessible everywhere)
*   **Protected:** `self._name` (Prefix with one underscore. Signals "please don't touch this outside the class or subclasses").
*   **Private:** `self.__name` (Prefix with two underscores. Python actually renames this behind the scenes—Name Mangling—to prevent access).

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner          # Public
        self._currency = "USD"      # Protected (Convention)
        self.__balance = balance    # Private (Name Mangled)

    def deposit(self, amount):
        self.__balance += amount    # Accessing private variable inside class
```

### C. Properties (Getters and Setters)
In C#, you love `{ get; set; }`. In Python, we use the `@property` decorator to achieve the exact same clean syntax.

**C# Equivalent:** `public double Salary { get; set; }`

**Python:**
```python
class Manager:
    def __init__(self, name, salary):
        self.name = name
        self.__salary = salary # Private

    # GETTER
    @property
    def salary(self):
        return self.__salary

    # SETTER
    @salary.setter
    def salary(self, value):
        if value < 0:
            raise ValueError("Salary cannot be negative")
        self.__salary = value

m = Manager("Alice", 90000)
print(m.salary)     # Calls getter
m.salary = 95000    # Calls setter
```

### D. Inheritance and Polymorphism
In C#, you use `:` to inherit and `base()` to call the parent constructor. In Python, you pass the parent class in parentheses and use `super()`. Python also doesn't require `virtual` and `override` keywords; methods are overridden simply by redefining them.

```python
# Parent Class
class User:
    def __init__(self, username, email):
        self.username = username
        self.email = email

    def login(self):
        print(f"{self.username} logged in.")

# Child Class inheriting from User
class Admin(User):
    def __init__(self, username, email, access_level):
        # Call Parent Constructor
        super().__init__(username, email) 
        self.access_level = access_level

    # Overriding the login method (Polymorphism)
    def login(self):
        print(f"ADMIN {self.username} logged in with level {self.access_level} clearance.")

admin = Admin("super_bob", "bob@corp.com", 5)
admin.login()
```

***

# 🎨 Lecture 2: Rapid UI Development with Streamlit (2 Hours)

As Full-Stack developers, you are used to building UIs with Angular. However, when developing AI Agents, writing an entire Angular frontend just to test a chatbot is a waste of time. 

**Streamlit** allows us to build modern web applications entirely in Python. 

## 1. Fundamental UI Elements
Streamlit reads code top-to-bottom. Here are the core building blocks.

```python
import streamlit as st

# Text Elements
st.title("Enterprise Dashboard")
st.header("Module 1")
st.subheader("Sub-section")
st.write("This is a standard paragraph.")

# Layouts (Columns)
col1, col2 = st.columns(2)
with col1:
    st.info("Information Card in Column 1")
with col2:
    st.warning("Warning Card in Column 2")

# Input Widgets
user_name = st.text_input("Enter your name:")
age = st.slider("Select your age:", 18, 65, 25)
department = st.selectbox("Department", ["HR", "IT", "Sales"])
```

## 2. Popups, Dialogs, and Notifications
Modern UIs need feedback mechanisms. Streamlit provides Toasts (snackbars) and Modals (dialog boxes).

```python
import streamlit as st

# 1. Toast Notification (Disappears after a few seconds)
if st.button("Save Profile"):
    st.toast("Profile saved successfully!", icon="✅")

# 2. Modal / Dialog Box (Requires Streamlit 1.34+)
@st.dialog("Delete Confirmation")
def delete_item_dialog(item_name):
    st.write(f"Are you sure you want to delete {item_name}?")
    reason = st.text_input("Reason for deletion:")
    if st.button("Confirm Delete"):
        st.session_state.deleted = True
        st.rerun() # Refreshes the app

if st.button("Delete User"):
    delete_item_dialog("John Doe")
```

## 3. Building a Chat Interface
Because we are building AI bots later in this course, understanding Streamlit's chat UI is critical.

```python
import streamlit as st

st.title("🤖 AI HR Assistant")

# Display previous messages (Hardcoded for this example)
st.chat_message("user").write("What is the maternity leave policy?")
st.chat_message("assistant").write("You are entitled to 84 working days of maternity leave.")

# Chat Input Box at the bottom of the screen
prompt = st.chat_input("Ask a question...")

if prompt:
    # Display what the user typed
    with st.chat_message("user"):
        st.write(prompt)
    
    # Display mock AI response
    with st.chat_message("assistant"):
        st.write("I am currently a mock bot. I will have an AI brain in Module 4!")
```

---

## 4. Complex UI Build: Leave Approval Tracker
Let's combine everything to build a visually appealing "Status Tracking UI". This uses columns, markdown, emojis, and avatars to replicate a real HR dashboard.

```python
import streamlit as st

st.set_page_config(layout="wide")
st.title("📅 Leave Request Tracker")

st.markdown("### Request ID: #REQ-9021")
st.write("**Employee:** Sarah Connor | **Type:** Annual Leave | **Days:** 3")

st.divider()

# Status Tracker UI using Columns
st.markdown("#### Approval Workflow Status")
c1, c2, c3 = st.columns(3)

with c1:
    st.success("✅ Submitted")
    st.caption("Oct 12, 09:00 AM")

with c2:
    st.success("✅ HR Reviewed")
    st.caption("Oct 12, 11:30 AM")

with c3:
    st.warning("⏳ Pending Manager")
    st.caption("Awaiting approval from Line Manager")

st.divider()

# Manager Action Area with Avatars
st.markdown("#### Manager Action Required")

# Using chat_message purely for the visual avatar effect
with st.chat_message("manager", avatar="🧑‍💼"):
    st.write("Please review the request and leave a comment.")
    
    comment = st.text_area("Manager Comment:", placeholder="Looks good to me...")
    
    btn_col1, btn_col2 = st.columns([1, 4])
    
    with btn_col1:
        if st.button("🟢 Approve"):
            st.toast("Leave Approved!", icon="🎉")
    
    with btn_col2:
        if st.button("❌ Reject"):
            st.toast("Leave Rejected.", icon="🛑")
```

---

### 📝 Lab Activity (For Students)
**Task:** Build the "Employee Onboarding Checklist UI".
1. Create a Python Class `NewHire` with properties: `name`, `role`, and a dictionary `checklist` containing tasks like `{"Laptop": False, "ID Card": False}`.
2. Use Streamlit to display a nice dashboard for the New Hire.
3. Use `st.checkbox()` elements to represent the checklist.
4. When all checkboxes are ticked, trigger a pop-up dialog (`@st.dialog`) congratulating the employee for completing onboarding!
