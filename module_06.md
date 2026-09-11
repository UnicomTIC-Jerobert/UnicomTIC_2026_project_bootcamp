Here is the complete **Mentor's Teaching Kit for Module 6**. 

You can copy this block, save it as `Module-6-Agentic-AI.md`, and push it to your GitHub repository.

***

# Module 6: Agentic AI & The C# Bridge

**Duration:** 4 Hours (1.5 hrs Lecture/Live Code + 2.5 hrs Lab)  
**Domain Context:** The Autonomous Leave Booker  
**Objective:** Transition from AI that just "reads" (RAG) to AI that "acts" (Agents). Learn how to give AI tools (Python functions) and teach it to make HTTP requests to your C# backend.

---

## 📖 Part 1: Lecture Notes (Concepts)

### 1. Chains vs. Agents
*   **Chains (What we did in Mod 4 & 5):** A predetermined path. User -> Prompt -> LLM -> Output. The AI has no choices.
*   **Agents (The Future):** We give the AI a "Brain" and a "Toolbox". 
    *   *User:* "What is my leave balance, and if I have enough, book tomorrow off."
    *   *AI Reasoning:* "First, I need to use the `CheckBalance` tool. Ah, they have 5 days. Next, I need to use the `BookLeave` tool."
    *   The AI decides *which* tool to use, *when* to use it, and *what* data to pass into it.

### 2. What is a LangChain `@tool`?
To an AI, a tool is just a standard Python function. But we must use the `@tool` decorator and provide a **Docstring** (the text inside `""" """`). 
**Crucial:** The AI doesn't read your Python code. It reads the Docstring to understand what the tool does. If your docstring is bad, the AI won't know how to use it!

### 3. The C# Bridge (`requests` library)
Your AI sidecar is written in Python, but your Database and Business Logic are inside your C# ASP.NET Core Monolith. 
How do they talk? 
Inside our Python `@tool`, we use the `requests` library to make standard `GET` and `POST` HTTP requests to your C# APIs, just like an Angular frontend would.

---

## 💻 Part 2: Live Code-Along (Follow the Mentor)
> **Mentor Note:** Ensure they have `requests` installed (`pip install requests langchain langchain-openai`). We will build a simple Agent that can check the weather using a mock tool.

**File:** `live_agent.py`
```python
from dotenv import load_dotenv
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate

load_dotenv()

# 1. Define the Tool (Notice the detailed Docstring!)
@tool
def get_employee_department(employee_name: str) -> str:
    """Useful for finding out which department an employee works in. 
    Pass the employee's first name as the argument."""
    
    print(f"--- 🛠️ AI IS USING THE TOOL FOR: {employee_name} ---")
    # In reality, this would be a DB call or API call.
    mock_db = {"john": "Engineering", "sarah": "Human Resources"}
    
    name_lower = employee_name.lower()
    return mock_db.get(name_lower, "Employee not found.")

# 2. Setup the AI Model and bind the tools to it
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
tools = [get_employee_department]

# 3. Create the Prompt and Agent
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful HR assistant. Use the provided tools to answer questions."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}") # This is where the AI writes its "thinking" notes
])

agent = create_tool_calling_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True) # verbose=True lets us see its brain!

# 4. Run the Agent
print("User: What department does Sarah work in?")
response = agent_executor.invoke({"input": "What department does Sarah work in?"})

print("\nFinal Answer to User:")
print(response["output"])
```

---

## 🧪 Part 3: Student Lab Activity (2.5 Hours)

**Scenario:** 
You are building the **Autonomous Leave Booker**. Employees will chat with a Streamlit interface. If they ask to book leave, the AI must autonomously extract the dates and trigger an HTTP POST request to the C# backend to save it in the database.

*(Since we don't have the C# backend running right now, we will simulate the C# backend using `https://httpbin.org/post`—a free developer API that simply echoes back whatever data you send it).*

**Your Tasks:**
1.  **Create the Bridge Tool:** 
    *   Write a Python function `book_leave_in_csharp(name: str, leave_type: str, date: str)`.
    *   Add the `@tool` decorator and a highly descriptive docstring.
    *   Inside the function, create a Python Dictionary containing the data.
    *   Use `requests.post("https://httpbin.org/post", json=your_data_dict)`.
    *   Return a success message from the function.
2.  **Setup the Agent:**
    *   Initialize `ChatOpenAI`.
    *   Create a prompt instructing the AI to act as a Leave Booking Assistant.
    *   Create the `AgentExecutor` with your tool.
3.  **Build the Streamlit Chat UI:**
    *   Create a text input for the user's message (e.g., *"Hi, I am Alex. Please book sick leave for me tomorrow."*).
    *   When the user clicks submit, pass the text to the `agent_executor`.
    *   Display the final response on the screen.

---

## 🔐 Part 4: Mentor's Answer Key (Solution)
> **Mentor Note:** The main stumbling block for students will be writing a good docstring. If the AI doesn't use the tool, tell the students to check their docstring! Also, point out how the AI handles extracting "Alex", "sick", and "tomorrow" perfectly to pass into the tool parameters.

**File:** `lab_solution.py`
```python
import streamlit as st
import requests
from dotenv import load_dotenv
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate

load_dotenv()

# --- 1. The Tool (The C# Bridge) ---
@tool
def book_leave_in_csharp(name: str, leave_type: str, date: str) -> str:
    """
    Use this tool whenever a user asks to book, apply, or schedule leave.
    It sends the leave request to the C# backend database.
    Inputs required:
    - name: The employee's name.
    - leave_type: The type of leave (e.g., sick, annual, casual).
    - date: The requested date or time frame.
    """
    
    # 1. Prepare the JSON payload exactly as the C# DTO expects it
    payload = {
        "EmployeeName": name,
        "LeaveType": leave_type,
        "LeaveDate": date
    }
    
    # 2. Make the HTTP POST request to the backend API (Simulated with httpbin)
    try:
        response = requests.post("https://httpbin.org/post", json=payload)
        
        if response.status_code == 200:
            return f"Successfully booked {leave_type} leave for {name} on {date} in the database."
        else:
            return "Error: Backend API rejected the request."
    except Exception as e:
        return f"Error connecting to backend: {str(e)}"


# --- 2. Agent Setup (Cached so it doesn't reload on every UI click) ---
@st.cache_resource
def setup_agent():
    llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
    tools = [book_leave_in_csharp]
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You are the company HR Leave Booking Assistant. Be polite. Use your tools to fulfill requests."),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}")
    ])
    
    agent = create_tool_calling_agent(llm, tools, prompt)
    return AgentExecutor(agent=agent, tools=tools, verbose=True)

agent_executor = setup_agent()

# --- 3. Streamlit UI ---
st.title("🤖 Autonomous Leave Booker")
st.write("Tell the AI what you need. It will trigger the backend API automatically.")

user_input = st.text_input("Message:")

if st.button("Send"):
    if user_input:
        with st.spinner("AI is thinking and acting..."):
            
            # Run the Agent
            response = agent_executor.invoke({"input": user_input})
            
            # Display output
            st.success("AI Response:")
            st.write(response["output"])
```

***

### End of Module 6
**Mentor Check-in:** Have the students look at their VS Code terminal (where `verbose=True` prints the AI's thoughts). Show them how the AI recognized the user's intent, stopped chatting, executed the Python function, waited for the HTTP response, and then generated the final text. 

*Tell them:* **"You have just built an Agentic AI system. This is the exact architecture you will use for your 3-month Capstone Projects!"**

*Next up in our Final Module 7: We will add the "Wow" factor. We will learn how to trigger Webhooks (Discord) to notify managers when the AI takes an action!*

---
*(Let me know when you are ready to generate the final **Module 7: Omnichannel & API Integrations**!)*
