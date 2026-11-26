# 🧁 Day 1: How LLMs Work & Your First API Call

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JAY4IGNITE/agenticAI/blob/main/DAY1/task.ipynb)

Comprehensive guide, concepts, and implementation details for **Day 1** of the Agentic AI Workshop.

---

## 🎯 Task Overview
Build your first customer-facing AI assistant for **Sweet Crumbs Home Bakery** using Groq API, LangChain `ChatGroq`, and Meta Llama Prompt Guard.

- **Primary Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY1/task.ipynb)
- **Domain:** Sweet Crumbs Home Bakery
- **Models Used:** 
  - `openai/gpt-oss-120b` (Chat completion & bakery assistant)
  - `meta-llama/llama-prompt-guard-2-22m` (Prompt injection & jailbreak safety classifier)

---

## 🧠 Key Conceptual Foundations

### 1. What an LLM Really Does
An LLM is not a database, search engine, or conscious intelligence; it is an **autoregressive next-token prediction engine** in a loop. Given a sequence of preceding tokens, it calculates probability distributions over vocabulary and samples the next likely token.

### 2. Tokens: The Unit of Everything
- **Analogy:** Lego bricks of language.
- $1 \text{ token} \approx 0.75 \text{ words}$ (or $\approx 4$ characters in English).
- Token count determines:
  1. API cost (prompt tokens + completion tokens).
  2. Latency (more completion tokens = more sequential inference passes).
  3. Context window consumption.

### 3. Temperature: The Creativity Dial
- **Low Temperature ($T \approx 0.0 - 0.2$):** Peaked probability distribution. Model picks top-ranked tokens deterministically. Used for pricing, inventory, policy compliance, and factual queries.
- **High Temperature ($T \approx 0.8 - 1.2$):** Flattens the distribution. Model samples lower-probability tokens for creative writing, marketing brainstorming, and tagline generation.

### 4. Statelessness (The Model Has No Memory)
Every API call to an LLM is completely isolated and independent. The model does not retain state between requests. Multi-turn chat requires client code to re-send the entire conversation history with each turn.

### 5. Context Window: The Size of the Desk
The fixed token limit an LLM can process in a single call (prompt + system prompt + output tokens). If the input exceeds the context window, tokens must be truncated or summarized.

### 6. Hallucinations & The 4-Pillar Prompt
LLMs generate text that sounds plausible, even when factually incorrect. To anchor responses, we engineer a **4-Pillar System Prompt**:
1. **ROLE:** Persona and identity (e.g., Sweet Crumbs assistant).
2. **FACTS:** Ground truth information (prices, 24-hour notice, eggless cakes).
3. **RULES:** Explicit policy boundaries (never promise delivery times; fallback to owner).
4. **FORMAT:** Output length and language constraints (under 50 words, polite, English/Telugu).

---

## 🏗️ Notebook Implementation Breakdown

### Cell 1: Environment & Dependency Installation
Installs required packages silently without pip version noise:
```python
!pip install -q --disable-pip-version-check langchain_groq groq python-dotenv ipywidgets
```

### Cell 2: Module Imports
Imports `ChatGroq` from LangChain and native `Groq` client:
```python
from langchain_groq import ChatGroq
from groq import Groq
```

### Cell 3: Multi-Tier API Key Resolution
Bulletproof authentication handling across local and cloud environments:
1. `os.environ.get("GROQ_API_KEY")`
2. Root `.env` discovery via `find_dotenv()` and relative path traversal.
3. Google Colab Secrets via `google.colab.userdata.get("GROQ_API_KEY")`.
4. Interactive masked fallback via `getpass.getpass(...)` preventing unhandled notebook crashes.

### Cell 5 & 6: 4-Pillar System Prompt & LangChain Invocation
```python
SYSTEM = """ROLE:   You are the assistant for Sweet Crumbs Home Bakery.
FACTS:  We bake to order. Orders need 24 hours' notice. All cakes have eggless versions.
        Plum cake Rs 500. Chocolate cake Rs 650. Cupcake Rs 80.
RULES:  Never promise a delivery time. If the answer isn't in FACTS, say "I'll check with the owner."
FORMAT: Under 50 words, friendly, English or Telugu matching the customer."""

def ask(question):
    return llm.invoke([("system", SYSTEM), ("human", question)]).content
```

### Cell 7: Verification Tests
- **Test 1:** Order notice rule check (`"Can I get my order tomorrow?"`).
- **Test 2:** Same-day eggless cake check (`"Can I get an eggless cake today evening?"`).

### Cell 8: Direct Groq API Call with Token Breakdown
Calls the native Groq completion API and prints token metrics:
```text
📊 Token Usage:
  Prompt Tokens:     186
  Completion Tokens: 161
  Total Tokens:      347
```

### Cell 9: Meta Llama Prompt Guard
Classifies adversarial inputs and prompt injections (`meta-llama/llama-prompt-guard-2-22m`):
- Score $\ge 0.5$: Unsafe / injection risk.
- Score $< 0.5$: Safe customer prompt.

### Cell 11: Interactive Dashboard
An `ipywidgets` GUI featuring:
- Live dropdown between **Bakery LLM** and **Prompt Guard**.
- Custom customer prompt text input.
- Preset test buttons:
  - *Sample 1:* Eggless cake notice
  - *Sample 2:* Cake prices inquiry
  - *Sample 3:* Prompt injection attack test

---

## 🚀 How to Run

### In Google Colab
1. Click the **Open In Colab** badge above.
2. In Colab, click the **🔑 Secrets** tab on the left sidebar.
3. Add a secret named `GROQ_API_KEY` with your Groq API key and toggle **Notebook access** to **ON**.
4. Run all cells (`Ctrl + F9` or `Runtime -> Run all`).

### Locally
1. Clone the repository:
   ```bash
   git clone https://github.com/JAY4IGNITE/24B11CS514_DriveReady_OPC_2026.git
   cd agenticAI
   ```
2. Copy `.env.example` to `.env` and insert your `GROQ_API_KEY`:
   ```bash
   cp .env.example .env
   ```
3. Open `DAY1/task.ipynb` in VS Code or JupyterLab and execute all cells.
