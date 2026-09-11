Here is the complete **Mentor's Teaching Kit for Module 5**. 

You can copy this block, save it as `Module-5-RAG-and-VectorDBs.md`, and push it to your GitHub repository.

***

# Module 5: RAG - Giving AI a Memory 

**Duration:** 4 Hours (1.5 hrs Lecture/Live Code + 2.5 hrs Lab)  
**Domain Context:** The HR Policy Assistant Chatbot  
**Objective:** Stop AI from hallucinating. Learn how to parse PDFs, create Embeddings, store them in a Vector Database (ChromaDB), and use Retrieval-Augmented Generation (RAG) to answer questions based *only* on the provided document.

---

## 📖 Part 1: Lecture Notes (Concepts)

### 1. The Limitation of Standard LLMs
Standard models (like ChatGPT) have two massive problems for enterprise use:
1.  **They hallucinate:** If they don't know the answer, they make it up.
2.  **They lack context:** They don't know your specific company's internal rules.

### 2. What is RAG? (Retrieval-Augmented Generation)
RAG is a technique where we fetch relevant information *before* we ask the AI a question. 
*   **Imagine this:** It’s an open-book exam. RAG is the process of finding the right page in the textbook, handing that page to the AI, and saying, *"Answer the student's question using ONLY this page."*

### 3. The 4 Steps of the RAG Pipeline
To make this work, we follow a strict data engineering pipeline:
1.  **Load:** Read the PDF (e.g., The Company HR Handbook).
2.  **Split (Chunking):** We can't send a 500-page PDF to the AI all at once. We split it into smaller "chunks" (e.g., 1000 characters per chunk).
3.  **Embed & Store:** We convert the text chunks into numbers (Embeddings) so the computer can understand the *meaning* of the text. We store these numbers in a **Vector Database** (like ChromaDB).
4.  **Retrieve & Generate:** When a user asks a question, we search the Vector DB for the most relevant chunk, attach it to our prompt, and ask the LLM to generate an answer.

---

## 💻 Part 2: Live Code-Along (Follow the Mentor)
> **Mentor Note:** Have students install the required libraries: `pip install langchain-chroma pypdf sentence-transformers`. (Note: We use `sentence-transformers` for free, local embeddings so students don't burn through OpenAI credits during testing).
> 
> *Preparation:* Create a simple text file named `company_rules.txt` in your project folder with this content: *"The company dress code is business casual. Employees get 15 days of annual leave. Free lunch is provided on Fridays."*

**File:** `live_rag.py`
```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_community.embeddings import HuggingFaceEmbeddings

print("1. Loading Document...")
loader = TextLoader("company_rules.txt")
docs = loader.load()

print("2. Splitting text into chunks...")
text_splitter = RecursiveCharacterTextSplitter(chunk_size=50, chunk_overlap=10)
splits = text_splitter.split_documents(docs)

print("3. Converting to Embeddings and saving to Vector DB...")
# Using free HuggingFace embeddings
embedding_model = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2") 
vectorstore = Chroma.from_documents(documents=splits, embedding=embedding_model)

print("4. Retrieving relevant info...")
question = "Do we get free food?"

# Search the DB for text similar to the question
retriever = vectorstore.as_retriever(search_kwargs={"k": 1}) # Fetch top 1 chunk
results = retriever.invoke(question)

print(f"\nUser asked: {question}")
print("Found this in the database:")
print(results[0].page_content)
```

---

## 🧪 Part 3: Student Lab Activity (2.5 Hours)

**Scenario:** 
The HR team is tired of answering the same policy questions every day. You have been provided with the official "Employee Handbook" (PDF). Your task is to build the **HR Policy Assistant Chatbot** using Streamlit and RAG.

**Your Tasks:**
1.  **Preparation:** Ask ChatGPT (or write it yourself) to generate a 2-page PDF titled `hr_policy.pdf`. Include rules about maternity leave, working hours (9 AM to 5 PM), and the remote work policy (allowed 2 days a week). Put this PDF in your project folder.
2.  **The Backend (RAG Engine):** 
    *   Create `rag_engine.py`.
    *   Use `PyPDFLoader` to load the PDF.
    *   Split the document using `RecursiveCharacterTextSplitter` (Try chunk_size=500).
    *   Store it in ChromaDB using `HuggingFaceEmbeddings`.
3.  **The Generation Prompt:**
    *   Initialize `ChatOpenAI` (or Gemini/Groq).
    *   Create a LangChain Prompt: *"You are an HR Assistant. Answer the user's question using ONLY the following context. If you don't know, say 'I don't know'. Context: {context} | Question: {question}"*
4.  **The UI (Streamlit):**
    *   Create a Streamlit UI with a text input for the user's question.
    *   When the user submits a question, run the RAG pipeline: retrieve the relevant context, pass it to the LLM, and display the AI's answer on the screen.

---

## 🔐 Part 4: Mentor's Answer Key (Solution)
> **Mentor Note:** This solution combines the Vector DB retrieval with the LLM generation in one script for simplicity in Streamlit. Ensure students understand how `{context}` is dynamically injected into the prompt.

**File:** `lab_solution.py`
```python
import streamlit as st
from dotenv import load_dotenv
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_chroma import Chroma
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

load_dotenv()

# --- 1. RAG Setup & Ingestion (Runs once) ---
@st.cache_resource # Streamlit decorator so we don't reload the PDF on every click
def setup_rag():
    # Load PDF
    loader = PyPDFLoader("hr_policy.pdf")
    docs = loader.load()
    
    # Chunking
    text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
    splits = text_splitter.split_documents(docs)
    
    # Vector DB
    embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
    vectorstore = Chroma.from_documents(documents=splits, embedding=embeddings)
    
    return vectorstore.as_retriever(search_kwargs={"k": 2}) # Get top 2 chunks

retriever = setup_rag()
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)

# --- 2. Streamlit UI ---
st.title("🤖 HR Policy Assistant")
st.write("Ask me anything about the company policies!")

user_question = st.text_input("Your Question:")

if st.button("Ask"):
    if user_question:
        with st.spinner("Searching company handbooks..."):
            
            # --- 3. Retrieval ---
            # Fetch the relevant chunks from ChromaDB
            retrieved_docs = retriever.invoke(user_question)
            
            # Combine the chunks into one big string
            context_text = "\n\n".join([doc.page_content for doc in retrieved_docs])
            
            # --- 4. Generation ---
            prompt = ChatPromptTemplate.from_template("""
            You are a helpful HR Assistant. 
            Answer the question using ONLY the provided context. 
            If the answer is not in the context, say "I'm sorry, I cannot find that in the policy handbook."
            
            Context:
            {context}
            
            Question: {question}
            """)
            
            # Create the chain
            chain = prompt | llm
            
            # Get the final answer
            response = chain.invoke({"context": context_text, "question": user_question})
            
            st.success(response.content)
            
            # Optional: Show the retrieved chunks so students see how it works under the hood
            with st.expander("See the chunks retrieved from the PDF"):
                st.write(context_text)
```

***

### End of Module 5
**Mentor Check-in:** Have the students test their chatbot with a question *not* in the PDF (e.g., "What is the capital of France?"). The AI should reply, "I cannot find that in the policy handbook." This proves they have successfully prevented LLM hallucinations! This is exactly how they will build the **Sri Lankan Gazette** and **University Career** apps.

*Next up in Module 6: We transition from AI that just "reads" to AI that "acts". We will introduce Agentic AI and LangChain Tools!*

---
*(Let me know when you are ready to generate **Module 6: Agentic AI & The C# Bridge**!)*
