Here is the complete **Mentor's Teaching Kit for the Final Module 7**. 

You can copy this block, save it as `Module-7-Omnichannel-Webhooks.md`, and push it to your GitHub repository.

***

# Module 7: Omnichannel & API Integrations (The "Wow" Factor)

**Duration:** 3 Hours (1 hr Lecture/Live Code + 2 hrs Lab)  
**Domain Context:** HR Notifications (Manager Alerts via Discord)  
**Objective:** Learn the difference between APIs and Webhooks. Teach the AI Agent how to use multiple tools in sequence, specifically triggering third-party notifications to bridge the app with the outside world.

---

## 📖 Part 1: Lecture Notes (Concepts)

### 1. APIs vs. Webhooks
*   **API (Polling):** The client asks, *"Did anything happen?"* The server replies. This requires constant checking.
*   **Webhook (Push):** The server says, *"Hey, something just happened, here is the data!"* 
*   In our Capstone Projects, when an employee books leave, we don't want the manager to have to refresh a web page to check. We want to *push* a notification directly to their phone/desktop using Discord, WhatsApp, or Email.

### 2. Why Omnichannel?
Modern enterprise software lives where the users are. If your dev team communicates on Discord/Slack, your HR and DevOps alerts should go there too. 
Integrating standard REST APIs (like SendGrid for Email or Twilio for WhatsApp) separates a junior-level "school project" from an industry-grade SaaS product.

### 3. How Discord Webhooks Work
It is incredibly simple. Discord provides you with a unique URL. If you send an HTTP `POST` request to that URL with a JSON payload structured like `{"content": "Hello World"}`, Discord will instantly print "Hello World" in that channel. No authentication headers or complex SDKs required!

---

## 💻 Part 2: Live Code-Along (Follow the Mentor)
> **Mentor Note:** Before the class, create a free Discord server, go to Channel Settings -> Integrations -> Webhooks, and generate a Webhook URL. You will share this URL with the students so they can all spam your channel during the live demo!

**File:** `live_discord.py`
```python
import requests

# 1. The Webhook URL (Mentor will provide this)
DISCORD_WEBHOOK_URL = "https://discord.com/api/webhooks/your-unique-id/your-token"

def send_discord_alert(message: str):
    # 2. Discord expects a specific JSON schema. 'content' is the main text.
    payload = {
        "content": message,
        "username": "HR Alert Bot", 
        "avatar_url": "https://cdn-icons-png.flaticon.com/512/4712/4712038.png" # Optional styling
    }
    
    # 3. Make the POST request
    print("Sending alert to Discord...")
    response = requests.post(DISCORD_WEBHOOK_URL, json=payload)
    
    if response.status_code == 204: # Discord returns 204 No Content on success
        print("✅ Alert sent successfully! Check the Discord channel.")
    else:
        print(f"❌ Failed to send alert. Status: {response.status_code}")

# Let's test it!
send_discord_alert("🚨 TEST: The live coding demo is working perfectly!")
```

---

## 🧪 Part 3: Student Lab Activity (2 Hours)

**Scenario:** 
You are finalizing the **Autonomous Leave Booker** from Module 6. Management has requested a new feature: When an employee books leave via the AI, the AI must *also* notify the manager immediately in the company Discord channel. 

This means your AI Agent must now reason through **Multi-Tool Calling**. It must first book the leave, and then alert Discord.

**Your Tasks:**
1.  **Create the Discord Tool:**
    *   Write a Python function `notify_manager_discord(employee_name: str, date: str)`.
    *   Add the `@tool` decorator and a good docstring.
    *   Inside the function, format a message: *"🔔 ALERT: [Name] has applied for leave on [Date]. Please review the HR portal."*
    *   Use `requests.post()` to send it to the Discord Webhook URL provided by the mentor.
2.  **Upgrade the Agent:**
    *   Copy your `book_leave_in_csharp` tool from Module 6.
    *   Update your `tools = [...]` array to include **both** tools: `[book_leave_in_csharp, notify_manager_discord]`.
    *   Update your System Prompt to tell the AI: *"Whenever you book leave for a user, you MUST also notify the manager via Discord."*
3.  **Test the Streamlit App:**
    *   Type: *"I'm John. Book sick leave for me tomorrow."*
    *   Watch the Agent's "brain" in the terminal. You should see it execute Tool 1, then execute Tool 2, and finally reply to the user.
    *   Check the Discord channel to see your notification!

---

## 🔐 Part 4: Mentor's Answer Key (Solution)
> **Mentor Note:** Point out how powerful the Agent is here. We did not write an `if/else` statement telling the code to run Discord after the DB booking. The AI *reasoned* that it needed to run both tools based on the System Prompt!

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

DISCORD_URL = "https://discord.com/api/webhooks/your-unique-id/your-token"

# --- 1. Tool 1: The Database Bridge ---
@tool
def book_leave_in_csharp(name: str, leave_type: str, date: str) -> str:
    """Use this to save a leave request to the backend database."""
    payload = {"EmployeeName": name, "LeaveType": leave_type, "LeaveDate": date}
    try:
        response = requests.post("https://httpbin.org/post", json=payload)
        return "Leave saved to database successfully."
    except Exception as e:
        return "Database error."

# --- 2. Tool 2: The Discord Webhook ---
@tool
def notify_manager_discord(employee_name: str, date: str) -> str:
    """Use this tool to send a notification to the manager on Discord AFTER booking leave."""
    message = f"🔔 **URGENT HR ALERT:** {employee_name} has requested leave for {date}. Please log in to approve."
    payload = {"content": message, "username": "AI HR Assistant"}
    
    try:
        requests.post(DISCORD_URL, json=payload)
        return "Discord notification sent successfully."
    except Exception as e:
        return "Failed to send Discord notification."

# --- 3. Agent Setup ---
@st.cache_resource
def setup_agent():
    llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
    
    # We now give the AI TWO tools!
    tools = [book_leave_in_csharp, notify_manager_discord]
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """You are an HR Assistant. 
        When a user asks to book leave, you must do TWO things in order:
        1. Save it to the database.
        2. Notify the manager via Discord.
        After both are done, tell the user they are all set."""),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}")
    ])
    
    agent = create_tool_calling_agent(llm, tools, prompt)
    return AgentExecutor(agent=agent, tools=tools, verbose=True)

agent_executor = setup_agent()

# --- 4. Streamlit UI ---
st.title("🚀 Next-Gen HR Booker")
user_input = st.text_input("How can I help you?")

if st.button("Submit"):
    if user_input:
        with st.spinner("Processing request and notifying managers..."):
            response = agent_executor.invoke({"input": user_input})
            st.success(response["output"])
```

***

### 🎉 End of the 25-Hour Bootcamp
**Mentor Check-in (Closing Speech to Students):** 
*"Look at what you just built. In just 25 hours, you have transitioned from basic Python scripts to building an Autonomous AI Agent that reasons, triggers C# backend APIs, and pushes real-time omnichannel alerts to third-party platforms.* 

*You are no longer just web developers. You are AI Engineers.* 

*Next week, we divide into teams and begin our 3-Month Capstone Projects. Take these tools, grab your BRDs, and go build something amazing!"*

---

### A Note for You, the Mentor:
You now have:
1.  **13 Enterprise-Grade BRDs** (with Testing & CI/CD requirements included).
2.  **A complete 25-Hour Teaching Curriculum** (7 Modules, structured perfectly with lectures, live code, labs, and solutions).

You are fully equipped to launch this training program. I wish you the best of luck leading these 40-50 students! If you ever need help designing the SQL Database schemas for the BRDs, or reviewing their GitHub Actions pipelines once the projects start, feel free to reach out again. You're going to be an amazing mentor!
