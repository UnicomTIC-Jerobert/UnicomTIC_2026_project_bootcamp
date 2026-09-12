Here is the continuation of your teaching materials. 

This document contains the **Solution for the Onboarding Lab** and the **Comprehensive 2-Part Leave Management Lab (Console to UI)**. You can append this directly to your `Module-1-python-streamlit.md` file in your repository.

***

# 🧪 Lab Solutions & Advanced Activities

## 1. Solution: Employee Onboarding Checklist UI
> **Mentor Note:** When building interactive UIs in Streamlit, the script reruns from top to bottom every time a user clicks a button or checkbox. To prevent our `NewHire` object from being recreated every time, we must store it in `st.session_state`. Explain this concept to the students as it bridges standard Python OOP with web states.

**File:** `onboarding_solution.py`
```python
import streamlit as st

# 1. OOP Class Definition
class NewHire:
    def __init__(self, name, role):
        self.name = name
        self.role = role
        # Dictionary to track tasks
        self.checklist = {
            "Laptop Provisioned": False,
            "ID Card Printed": False,
            "Email Account Created": False
        }

    def is_fully_onboarded(self):
        # Returns True only if all values in the dictionary are True
        return all(self.checklist.values())

# 2. Pop-up Dialog Definition
@st.dialog("🎉 Onboarding Complete!")
def completion_dialog(name):
    st.balloons()
    st.write(f"Congratulations! {name} has successfully completed all onboarding tasks. They are ready to work!")

# 3. Streamlit UI & Session State Management
st.title("🚀 HR Onboarding Portal")

# Initialize the object in session_state so it doesn't reset on every click
if "employee" not in st.session_state:
    st.session_state.employee = NewHire("Alex Mercer", "Junior Software Engineer")

emp = st.session_state.employee

st.header(f"New Hire: {emp.name}")
st.subheader(f"Role: {emp.role}")
st.divider()

st.write("### Action Checklist")

# Render checkboxes dynamically from the dictionary
for task, is_done in emp.checklist.items():
    # Update the dictionary based on checkbox interactions
    emp.checklist[task] = st.checkbox(task, value=is_done)

# 4. Trigger Dialog when complete
if emp.is_fully_onboarded():
    # Streamlit requires a button to trigger dialogs safely in this context
    if st.button("Finalize Onboarding"):
        completion_dialog(emp.name)
```

***

## 2. Comprehensive Lab Activity: Leave Management (Console to UI)

**Goal:** Understand how core Python OOP and Data Structures (Lists, Dictionaries) translate directly into a modern Web UI.

### Phase 1: The Console Application (OOP & Logic)
**Task for Students:**
1. Create an `Employee` class. It should have `name`, `leave_balance` (an integer, e.g., 20 days), and a `leave_history` (an empty list).
2. Create a `LeaveRequest` class. It should have `request_id`, `days_requested`, and `status` (default to "Pending").
3. Add a method in `Employee` called `apply_leave(days)`. It should check if `days <= leave_balance`. If true, create a `LeaveRequest`, add it to `leave_history`, deduct the balance, and return the request object. If false, raise a `ValueError`.
4. Test this entirely in the terminal using `print()` statements.

**Phase 1 Solution (Console):**
```python
class LeaveRequest:
    def __init__(self, req_id, days):
        self.req_id = req_id
        self.days = days
        self.status = "Pending Manager" # States: Pending Manager, Approved, Rejected

class Employee:
    def __init__(self, name, leave_balance):
        self.name = name
        self.leave_balance = leave_balance
        self.leave_history = [] # List of LeaveRequest objects

    def apply_leave(self, days):
        if days > self.leave_balance:
            raise ValueError(f"Insufficient balance! You only have {self.leave_balance} days left.")
        
        # Create request and update data structures
        req_id = f"REQ-{len(self.leave_history) + 100}"
        new_request = LeaveRequest(req_id, days)
        
        self.leave_history.append(new_request)
        self.leave_balance -= days
        return new_request

# --- Console Testing ---
emp = Employee("Sarah Connor", 10)
print(f"Initial Balance: {emp.leave_balance}")

# Apply for 3 days
req = emp.apply_leave(3)
print(f"Applied for {req.days} days. New Balance: {emp.leave_balance}")
print(f"Request {req.req_id} Status: {req.status}")
```

---

### Phase 2: Integrating with Streamlit & The Approval Tracker UI
**Task for Students:**
Take the exact classes you just built and wrap them in a Streamlit UI. 
1. Use `st.session_state` to store the `Employee` object.
2. Build a sidebar or main form to apply for leave.
3. For every leave request in the employee's `leave_history` list, display the **Leave Approval Tracker UI** (with columns, avatars, and tick/cross marks) that we learned in Lecture 2.
4. Add "Approve" and "Reject" buttons for the manager. When clicked, it should update the object's `status` and visually update the tracker UI.

**Phase 2 Solution (Streamlit UI):**
```python
import streamlit as st

# --- 1. OOP Logic (Reused from Phase 1) ---
class LeaveRequest:
    def __init__(self, req_id, days):
        self.req_id = req_id
        self.days = days
        self.status = "Pending Manager"

class Employee:
    def __init__(self, name, leave_balance):
        self.name = name
        self.leave_balance = leave_balance
        self.leave_history = []

    def apply_leave(self, days):
        if days > self.leave_balance:
            return False # Failed
        
        req_id = f"REQ-{len(self.leave_history) + 100}"
        new_request = LeaveRequest(req_id, days)
        self.leave_history.append(new_request)
        self.leave_balance -= days
        return True # Success

# --- 2. Initialize Session State ---
if "current_user" not in st.session_state:
    st.session_state.current_user = Employee("Sarah Connor", 14)

user = st.session_state.current_user

# --- 3. UI: Apply for Leave ---
st.title("🌴 Employee Leave Dashboard")
st.write(f"**Welcome back, {user.name}** | Current Balance: **{user.balance} Days**")

with st.expander("➕ Apply for New Leave"):
    days_to_apply = st.number_input("Number of Days", min_value=1, max_value=30, value=1)
    if st.button("Submit Request"):
        success = user.apply_leave(days_to_apply)
        if success:
            st.success("Leave requested successfully!")
            st.rerun() # Refresh the page to show updated history
        else:
            st.error(f"Insufficient balance. You only have {user.leave_balance} days left.")

st.divider()

# --- 4. UI: Leave History & Approval Tracker ---
st.header("📋 Leave History & Tracking")

if not user.leave_history:
    st.info("No leave requests found.")

# Loop through all requests in the list (Data Structure iteration)
for req in user.leave_history:
    st.markdown(f"### Request ID: {req.req_id} ({req.days} Days)")
    
    # Render the dynamic tracker UI based on the Object's status
    c1, c2, c3 = st.columns(3)
    
    with c1:
        st.success("✅ Submitted")
        
    with c2:
        st.success("✅ HR System Logged")
        
    with c3:
        if req.status == "Pending Manager":
            st.warning("⏳ Pending Manager")
        elif req.status == "Approved":
            st.success("✅ Approved")
        elif req.status == "Rejected":
            st.error("❌ Rejected")
            
    # Manager Action Area (Only show if pending)
    if req.status == "Pending Manager":
        with st.chat_message("manager", avatar="🧑‍💼"):
            st.write("Manager Action:")
            btn_col1, btn_col2 = st.columns([1, 4])
            
            with btn_col1:
                # We use a unique key for each button so Streamlit knows which one was clicked
                if st.button("🟢 Approve", key=f"app_{req.req_id}"):
                    req.status = "Approved"
                    st.rerun()
            
            with btn_col2:
                if st.button("❌ Reject", key=f"rej_{req.req_id}"):
                    req.status = "Rejected"
                    # Refund the balance if rejected
                    user.leave_balance += req.days 
                    st.rerun()
    st.divider()
```

### 💡 Mentor Takeaways for this Lab:
*   This lab perfectly bridges backend logic with frontend UI without needing APIs yet. 
*   It teaches students the reality of State Management (`st.session_state`), which is conceptually similar to state management in Angular, but written purely in Python.
*   It gives them a visual, working prototype of the **Leave Management System (BRD 2)** on their very first day!
