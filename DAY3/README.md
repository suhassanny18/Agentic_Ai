# 🧩 Day 3: Prompt Templates, Memory, Evaluation & Agent Sneak Peek

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JAY4IGNITE/24B11CS514_DriveReady_OPC_2026/blob/main/DAY3/task.ipynb)

Comprehensive guide, concepts, and implementation details for **Day 3** of the Agentic AI Workshop — adapted to use the **Google Gemini API** (`langchain-google-genai`).

---

## 🎯 Task Overview
Refactor raw API calls into composable **LangChain LCEL chains** with `ChatPromptTemplate`, `MessagesPlaceholder`, and `StrOutputParser`, evaluate prompt quality systematically with a CSV test suite, and preview LangChain's `create_agent`.

- **Primary Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY3/task.ipynb)
- **Test Suites:**
  - [`tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/tests.csv) — Sweet Crumbs Home Bakery (10 test cases)
  - [`brightpage_tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/brightpage_tests.csv) — BrightPage Stationery & Office Supplies (10 test cases)
- **Model Used:** `gemini-3.5-flash-lite` via Google Gemini API (`ChatGoogleGenerativeAI`)

---

## 🧠 Core Engineering Concepts

### 1. Composable LCEL Chain (`prompt | llm | StrOutputParser()`)
Instead of manually assembling message lists and extracting `.content` every time:
- **`ChatPromptTemplate.from_messages(...)`**: Structures `system`, conversation `history`, and `human` inputs cleanly.
- **`MessagesPlaceholder("history")`**: Injects multi-turn conversation history directly into the template.
- **`StrOutputParser()`**: Converts the model's `AIMessage` output into a plain Python `str`.

### 2. Automated Prompt Evaluation (`tests.csv`)
Testing prompts manually with 1–2 chats misses regressions. A CSV test suite pairs realistic customer `question` inputs with deterministic `must_include` assertions so every prompt change yields a measurable pass rate (`passed / total`).

### 3. Temperature & Determinism (`0.3` vs `1.0`)
- **`temperature=0.3`**: Keeps factual prices, policies, and fallback phrasing consistent across runs.
- **`temperature=1.0`**: Increases response variety, which can cause wording drift across identical test runs.

### 4. Agent Sneak Peek (`create_agent`)
`from langchain.agents import create_agent` wraps the model, system prompt, and tools into an agent loop. With `tools=[]` it behaves like a standard chain; adding tools (e.g., inventory or price lookup) turns it into a multi-step reasoning agent.

### 5. Choosing the Right Tool: Plain Code vs One LLM Call vs Agent
| Business Job (BrightPage Stationery) | Approach | Why |
| :--- | :--- | :--- |
| Calculate cart subtotal, 10% bulk discount, and ₹40 delivery fee | **Plain code** | Deterministic arithmetic — exact, free, and instant. |
| Classify customer message as `order`, `inquiry`, or `complaint` | **One LLM call** | Natural language classification needs understanding, not tool loops. |
| Check stock, compute bulk totals, and log an order end-to-end | **Agent** | Requires multi-step reasoning and deciding which tools to call. |

---

## 🏗️ Notebook Implementation Breakdown

1. **Cell 1 · Setup (Gemini API):** Installs `langchain` and `langchain-google-genai`, resolves `GOOGLE_API_KEY` (from Colab Secrets, `.env`, or prompt), and initializes `ChatGoogleGenerativeAI(model="gemini-3.5-flash-lite", temperature=0.3)`.
2. **Cell 2 · Template + Memory + Chain:** Builds the Sweet Crumbs assistant chain and `ask()` helper with multi-turn memory.
3. **Customer Verification:** Tests eggless options, same-day cake notice, local delivery rules, and 6-cupcake math.
4. **Cell 3 · Automated Evaluation (`tests.csv`):** Runs `run_tests("tests.csv")` and reports pass/fail per row plus overall pass rate.
5. **Cell 4 · Agent Sneak Peek:** Runs `create_agent(model=llm, tools=[], system_prompt=SYSTEM)`.
6. **Your 30% (BrightPage Stationery & Office Supplies):**
   - Rewrites `SYSTEM` for BrightPage's 10-item catalogue and policies.
   - Evaluates against `brightpage_tests.csv` (10 customer questions).
   - Improves the prompt (`V1` → `V2`) and compares pass rates.
   - Switches to `temperature=1.0` and runs tests twice to compare output stability.
   - Maps 3 business jobs to `plain code`, `one LLM call`, or `agent`.

---

## 🚀 How to Run

### In Google Colab
1. Open [`DAY3/task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY3/task.ipynb) in Google Colab.
2. Click **🔑 Secrets** on the left sidebar, add **`GOOGLE_API_KEY`** with your key from [Google AI Studio](https://aistudio.google.com/app/apikey), and toggle **Notebook access** to **ON**.
3. Run all cells (`Ctrl + F9`).

### Locally
1. Add `GOOGLE_API_KEY=your_gemini_api_key_here` to the root [`.env`](file:///c:/Users/ramuv/AgenticAI/.env) file.
2. Install dependencies: `pip install langchain langchain-google-genai python-dotenv`.
3. Open `DAY3/task.ipynb` and run all cells.
