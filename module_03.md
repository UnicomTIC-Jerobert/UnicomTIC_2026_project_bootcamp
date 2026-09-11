Here is the complete **Mentor's Teaching Kit for Module 3**. 

You can copy this block, save it as `Module-3-QA-and-Automation.md`, and push it to your GitHub repository.

***

# Module 3: Quality Assurance & UI Automation

**Duration:** 4 Hours (1.5 hrs Lecture/Live Code + 2.5 hrs Lab)  
**Domain Context:** Testing the Payroll API and the Leave UI  
**Objective:** Learn the Testing Pyramid. Prove your code works by writing Unit Tests (`pytest`), API Tests (Postman), and UI Automation scripts (Selenium).

---

## 📖 Part 1: Lecture Notes (Concepts)

### 1. The Testing Pyramid
In enterprise software, you cannot just say "it works on my machine." You must write code that *tests* your code.
*   **Unit Tests (Bottom Layer):** Tests individual functions in isolation (e.g., does the tax math work?). These are written in Python using `pytest`. They are fast and run in milliseconds.
*   **API / Integration Tests (Middle Layer):** Tests the HTTP endpoints (e.g., does `POST /api/payroll` return a `200 OK`?). We use **Postman** for this.
*   **UI / End-to-End Tests (Top Layer):** Simulates a real human clicking buttons on the screen. We use **Selenium**. These are slower but test the whole system.

### 2. Introduction to `pytest`
*   `pytest` is the industry standard for Python testing. 
*   **The Rule:** Your test file must start with `test_` (e.g., `test_logic.py`). Your test functions must also start with `test_`.
*   You use the `assert` keyword to check if the result matches your expectation.

### 3. Introduction to Selenium (UI Automation)
*   Selenium uses a "WebDriver" to literally take control of your Google Chrome browser.
*   It finds elements on the web page using HTML tags, IDs, or XPath, and can type text (`send_keys`) or click buttons (`click()`).

---

## 💻 Part 2: Live Code-Along (Follow the Mentor)
> **Mentor Note:** Have students install testing libraries: `pip install pytest selenium webdriver-manager`. 

### A. Live Pytest Example
**File:** `test_demo.py`
```python
# 1. The function we want to test
def calculate_bonus(salary, rating):
    if rating >= 5:
        return salary * 0.10
    return 0

# 2. The Unit Test
def test_calculate_bonus_high_rating():
    # Arrange
    base_salary = 1000
    perf_rating = 5
    
    # Act
    result = calculate_bonus(base_salary, perf_rating)
    
    # Assert (We expect a $100 bonus)
    assert result == 100
```
*Run it by typing `pytest` in the terminal.*

### B. Live Selenium Example
**File:** `live_selenium.py`
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time

# 1. Open Google Chrome autonomously
driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))

# 2. Navigate to a webpage
driver.get("https://www.wikipedia.org/")

# 3. Find the search box by its HTML 'name' attribute and type something
search_box = driver.find_element(By.NAME, "search")
search_box.send_keys("Python (programming language)")

# 4. Find the search button and click it
submit_button = driver.find_element(By.CSS_SELECTOR, "button[type='submit']")
submit_button.click()

# Keep browser open for 3 seconds so students can see it, then close
time.sleep(3)
driver.quit()
```

---

## 🧪 Part 3: Student Lab Activity (2.5 Hours)

**Scenario:** 
You need to prove to the QA Manager that the code you wrote in Modules 1 and 2 actually works. You will write a Unit Test for the Payroll math, an API test in Postman, and a Selenium script that automates the Leave UI.

**Your Tasks:**
1.  **Unit Testing the Payroll Math (pytest):**
    *   Create a file `test_payroll.py`.
    *   Write a pure Python function `calculate_net_pay(base, ot_hours)` containing the math from Module 2 (OT = $20/hr, Tax = 10%).
    *   Write a `pytest` function that asserts: If Base is `1000` and OT is `10`, the gross is `1200`, tax is `120`, and net pay should exactly equal `1080`.
2.  **API Testing (Postman):**
    *   Start your FastAPI server from Module 2 (`uvicorn`).
    *   Open Postman. Create a new `POST` request to `http://127.0.0.1:8000/api/payroll/calculate`.
    *   Add the JSON payload in the "Body" tab. Send it, and verify you get a `200 OK` and the correct JSON response.
3.  **UI Automation (Selenium):**
    *   Start your Streamlit Leave Portal from Module 1 (`streamlit run`). It runs on `http://localhost:8501`.
    *   Write a Python script `test_ui.py` using Selenium.
    *   Make Selenium open `http://localhost:8501`.
    *   *Challenge:* Have Selenium find the text input, type a name, and click the "Submit Request" button automatically.

---

## 🔐 Part 4: Mentor's Answer Key (Solution)
> **Mentor Note:** For Task 2, just walk around and visually verify they have 200 OK in Postman. For Task 3, Streamlit elements don't have standard IDs, so we use `xpath` or generic HTML tags.

### Solution 1: Pytest (`test_payroll.py`)
```python
# The business logic isolated from the API
def calculate_net_pay(base_salary, ot_hours):
    gross_pay = base_salary + (ot_hours * 20.0)
    tax = gross_pay * 0.10
    return gross_pay - tax

# The test
def test_payroll_calculation():
    # Arrange
    base = 1000
    ot = 10
    
    # Act
    net = calculate_net_pay(base, ot)
    
    # Assert
    assert net == 1080.0
```

### Solution 3: Selenium UI Test (`test_ui.py`)
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time

# Start Chrome
driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))

try:
    # 1. Open the local Streamlit App
    driver.get("http://localhost:8501")
    
    # Streamlit apps take a second to render
    time.sleep(3)
    
    # 2. Find the Name input box (Streamlit uses standard <input> tags for text)
    # Using XPath to find the first text input on the screen
    name_input = driver.find_element(By.XPATH, "//input[@type='text']")
    name_input.send_keys("Automated QA Tester")
    time.sleep(1) # Just to watch it happen
    
    # 3. Find the Submit Button (Streamlit buttons have a specific class or we can find by text)
    # Finding the button containing the text "Submit Request"
    submit_btn = driver.find_element(By.XPATH, "//button[contains(., 'Submit Request')]")
    submit_btn.click()
    
    # Wait to see the success message render
    time.sleep(3)
    
    print("✅ UI Automation Test Passed Successfully!")

finally:
    driver.quit()
```

***

### End of Module 3
**Mentor Check-in:** Explain that this is what sets Juniors apart from Seniors. Seniors don't just write code; they write code that *tests* code. 
*Next up in Module 4: We will finally introduce AI. We will step away from standard APIs and connect to LangChain to parse messy human text!*

---
*(Let me know when you are ready to generate **Module 4: AI Fundamentals & LangChain**!)*
