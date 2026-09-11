Here is the complete **Mentor's Teaching Kit for Module 4**. 

You can copy this block, save it as `Module-4-LangChain-and-AI.md`, and push it to your GitHub repository.

***

# Module 4: AI Fundamentals & LangChain

**Duration:** 3 Hours (1 hr Lecture/Live Code + 2 hrs Lab)  
**Domain Context:** HR Intent Parsing (Reading messy text)  
**Objective:** Move away from web-interface ChatGPT. Learn to integrate AI programmatically using LangChain and force it to return strict JSON data that our APIs can understand.

---

## 📖 Part 1: Lecture Notes (Concepts)

### 1. What is LangChain?
When you use ChatGPT on your browser, you type text and get text back. But in Enterprise Software, we don't want conversational text. We want the AI to process data, connect to databases, and return structured JSON.
*   **LangChain** is the industry-standard Python framework that acts as the "glue" between your code, your data, and the AI models (OpenAI, Gemini, Llama, etc.).

### 2. System Messages vs. Human Messages
In LangChain, we don't just send one big string to the AI. We structure the prompt:
*   **System Message:** The hidden instructions defining the AI's persona and rules. *(e.g., "You are an HR Assistant. Never answer questions outside of HR topics.")*
*   **Human Message:** The actual input from the user. *(e.g., "How do I book leave?")*

### 3. The Power of Output Parsing
This is the most important concept today. Your C# backend **cannot** read a response like: *"Sure! It looks like you want to take sick leave today."* It will crash. 
Your C# backend needs: `{"intent": "leave_request", "type": "sick", "date": "today"}`. 
We will use LangChain to force the AI to strip away all polite conversation and only output JSON.

---

## 💻 Part 2: Live Code-Along (Follow the Mentor)
> **Mentor Note:** Have students install LangChain and the OpenAI SDK: `pip install langchain langchain-openai python-dotenv`.  
> *Note on API Keys:* You will need an OpenAI API key (or Groq/Gemini if you are using free tiers). Ensure students know how to create a `.env` file for security.

**File:** `.env`
```text
OPENAI_API_KEY=sk-your-api-key-here
```

**File:** `live_ai.py`
```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# 1. Load the API key from the .env file
load_dotenv()

# 2. Initialize the AI Model (Temperature 0 means strictly logical, no creative hallucinations)
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)

# 3. Create our Prompt Template (System + Human)
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an expert HR assistant. Extract the job title the user is asking about. ONLY reply with the job title, nothing else."),
    ("human", "{user_input}")
])

# 4. Chain them together using LCEL (LangChain Expression Language)
chain = prompt | llm

# 5. Run the AI
user_text = "Hi there, I was wondering if you are currently hiring any Senior Software Engineers for the backend team?"
print("Thinking...")

response = chain.invoke({"user_input": user_text})

print("--- AI Output ---")
print(response.content) # Output will strictly be: "Senior Software Engineer"
```

---

## 🧪 Part 3: Student Lab Activity (2 Hours)

**Scenario:** 
Employees often send messy text messages to the HR system like, *"Hey, I'm feeling sick today, I won't be able to come to work"* or *"I want to go on vacation next week for 3 days."* 

Your C# backend needs this structured. You must build a **FastAPI Endpoint** that uses LangChain to parse the messy text into a clean JSON object.

**Your Tasks:**
1.  **Setup FastAPI:** Create `ai_parser_api.py`. Define a Pydantic model `LeaveTextRequest` that accepts a `message` (string).
2.  **Setup LangChain:** Inside your API endpoint, initialize `ChatOpenAI`. 
3.  **Prompt Engineering:** Write a highly specific `System Message`. Tell the AI:
    *   "You extract leave intentions from employee messages."
    *   "You must extract the `leave_type` (sick, annual, or casual)."
    *   "You must extract the `date`."
    *   "You must reply ONLY in a valid JSON format like: `{\"intent\": \"leave\", \"leave_type\": \"sick\", \"date\": \"today\"}`. Do not include markdown formatting or polite text."
4.  **Execute & Return:** Run the chain with the user's message, take the AI's text response, convert it to a Python Dictionary using `json.loads()`, and return it from the FastAPI endpoint.
5.  **Test:** Run `uvicorn` and test your endpoint in the Swagger UI (`/docs`).

---

## 🔐 Part 4: Mentor's Answer Key (Solution)
> **Mentor Note:** The trick here is using Python's built-in `json` library to convert the AI's string output into an actual JSON response for the API. Remind them that Prompt Engineering is just as important as writing Python code.

**File:** `lab_solution.py`
```python
import json
from fastapi import FastAPI
from pydantic import BaseModel
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# Load environment variables (API Keys)
load_dotenv()

app = FastAPI(title="AI Intent Parser API")

# Request Model
class LeaveTextRequest(BaseModel):
    message: str

@app.post("/api/ai/parse-leave")
def parse_leave_message(request: LeaveTextRequest):
    
    # 1. Initialize AI Model
    llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
    
    # 2. Strict Prompt Engineering
    system_instruction = """
    You are an HR intent extraction engine.
    Extract the following from the user's message:
    1. leave_type (classify as: sick, annual, casual, or unknown)
    2. date (extract the date or time frame mentioned)
    
    You MUST respond ONLY with a raw JSON object. Do not use code blocks (```json). No conversational text.
    Format: {"intent": "leave_request", "leave_type": "...", "date": "..."}
    """
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", system_instruction),
        ("human", "{message}")
    ])
    
    # 3. Create and run the chain
    chain = prompt | llm
    ai_response = chain.invoke({"message": request.message})
    
    # 4. Parse the AI's string response into actual JSON
    try:
        structured_data = json.loads(ai_response.content)
        return structured_data
    except json.JSONDecodeError:
        return {"error": "AI failed to return valid JSON", "raw_output": ai_response.content}

```

***

### End of Module 4
**Mentor Check-in:** Have the students test their Swagger UI with different, weirdly phrased sentences (e.g., *"My head hurts, taking tomorrow off"*). Show them how the AI elegantly normalizes it into `{"leave_type": "sick", "date": "tomorrow"}`. Explain that they have just built the core logic for the AI sidecar!

*Next up in Module 5: We will tackle RAG (Retrieval-Augmented Generation). We will teach the AI how to read a PDF so it can answer questions based on your specific Company Policies!*

---
*(Let me know when you are ready to generate **Module 5: RAG - Giving AI a Memory**!)*
