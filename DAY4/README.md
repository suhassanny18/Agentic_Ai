# 🛠️ Day 4: Tools, Structured Records, Sheets & Agent Traces

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JAY4IGNITE/24B11CS514_DriveReady_OPC_2026/blob/main/DAY4/task.ipynb)

Comprehensive guide, concepts, and verified implementation details for **Day 4** of the Agentic AI Workshop.

---

## 🎯 Task Overview
Upgrade from tool-less chains to **tool-calling agents** using LangChain `create_agent`, `@tool` decorators, Pydantic schemas (`Order`, `StationeryOrder`), `with_structured_output`, and persistent order logging via Google Sheets (`gspread` in Colab) or a local CSV-backed `LocalWorksheet` mirror.

- **Primary Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY4/task.ipynb)
- **Domains:**
  - **Sweet Crumbs Home Bakery** (Workshop core + customer edge-case testing + `tool1` break-it experiment)
  - **BrightPage Stationery & Office Supplies** (Your 30%: 2 `@tool`s, `StationeryOrder` structured extraction, sheet & `log_stationery_order`)
- **Regression Test Suite:** [`../DAY3/tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/tests.csv) (expanded from 10 to 13 rows)
- **Local Sheet Outputs:** [`sweet_crumbs_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY4/sweet_crumbs_orders.csv), [`brightpage_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY4/brightpage_orders.csv)
- **Model Used:** `openai/gpt-oss-20b` via Groq (`ChatGroq`, `temperature=0.2`)

---

## 🧠 Core Engineering Concepts

### 1. `@tool` Functions & Why Docstrings Are the Model's API Docs
- Tools expose Python functions to the LLM with typed arguments (`item: str`, `qty: int = 1`) and clear Google-style docstrings.
- **Break-It Experiment (`tool1` / `"does stuff"`):** Renaming `get_price` to `tool1` with docstring `"does stuff"` caused the model to make **0 tool calls** (`[]`) on `"How much are 3 plum cakes?"` and ask for order details instead of quoting a price. Restoring the descriptive name `get_price` and clear docstring immediately restored tool selection (`['get_price']` -> `3 x plum cake = Rs 1500`).

### 2. Pydantic Schemas & `with_structured_output`
- **Sweet Crumbs (`Order`):** Validates `name`, `item` (`Literal["plum cake", "chocolate cake", "cupcake"]`), `qty >= 1`, `date`, and `eggless`.
- **BrightPage (`StationeryOrder`):** Validates `customer_name`, `item` (10-item `Literal`), `qty >= 1`, `delivery_type` (`"home_delivery"` | `"store_pickup"`), `delivery_date`, and optional `notes`.
- Tested `llm.with_structured_output(StationeryOrder)` on **5 messy WhatsApp-style customer messages** (slang, typos, abbreviations, pickup vs delivery, and notes) — all 5 parsed cleanly into valid `StationeryOrder` objects.

### 3. Reading Agent Traces with `pretty_print()`
For `"Are you open on Sunday? How much are 2 chocolate cakes?"`:
- **Model calls (`AIMessage`):** `3`
- **Tool calls requested:** `2` (`['is_open', 'get_price']`)
- **Tool results (`ToolMessage`):** `2`

Execution sequence:
1. `HumanMessage` -> `AIMessage` calls `is_open(day="sunday")` -> `ToolMessage` returns `"Open 10 am to 6 pm on sunday."`
2. `AIMessage` calls `get_price(item="chocolate cake", qty=2)` -> `ToolMessage` returns `"2 x chocolate cake = Rs 1300"`
3. Final `AIMessage` synthesizes both tool results into a concise customer reply.

### 4. Customer Edge-Case Testing & Sheet Integrity
Tested three adversarial/incomplete customer queries on Sweet Crumbs:
1. **Unlisted item (`"donut"`):** `"Can I order 2 donuts for tomorrow?"` -> Explains donuts are not on the menu and lists available items.
2. **Missing pickup date:** `"Hi, I'm Sita. Please book 2 plum cakes."` -> Asks for pickup date (and eggless preference) without calling `log_order`.
3. **Invalid quantity (`0 cupcakes`):** `"Hi, I'm Kiran. Please order 0 cupcakes for 25 Dec."` -> Explains minimum quantity is at least 1 and asks how many cupcakes are needed.

- **Sheet integrity verified:** Before and after all 3 edge tests, the sheet retained only the 1 valid order (`Ravi`, `chocolate cake`, `2`, `24 Dec`, `yes`, `1300`).
- **Day 3 `tests.csv` evaluation:** Added these 3 cases as rows 11–13 in [`DAY3/tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/tests.csv); the Day 4 Sweet Crumbs agent passed **13/13 (100%)**.

---

## 🏗️ Notebook Implementation Breakdown

1. **Section 1 · Setup & Sheet Connection:** Loads `GROQ_API_KEY` from `.env` (local), Colab Secrets, or `getpass`, defines `invoke_retry` for transient errors, and provides `init_sheet()` using `gspread` in Google Colab or `LocalWorksheet` locally.
2. **Section 2 · Sweet Crumbs Bakery Agent:** Defines `Order`, `@tool`s (`get_price`, `is_open`, `log_order`), and `create_agent`, verifying multi-turn conversation and Ravi's booked order in the sheet.
3. **Section 3 · Read the Trace:** Prints the full `pretty_print()` trace and counts `AIMessage` model calls (`3`) vs `ToolMessage`/tool calls (`2`).
4. **Section 4 · Test It Like a Customer:** Runs the `"donut"`, missing-date, and `0 cupcakes` checks, asserts no bad rows entered the sheet, and evaluates all 13 rows of [`DAY3/tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/tests.csv) (**13/13 passed**).
5. **Section 5 · Your 30% (BrightPage Stationery & Office Supplies):**
   - Converts 2 Day-2 business functions into `@tool`s: `get_stationery_price` (with 10% bulk discount on `qty >= 10` or subtotal `> Rs 1000`) and `check_store_and_delivery` (hours + free delivery above `Rs 500`, `Rs 40` otherwise).
   - Tests `StationeryOrder` with `with_structured_output` on 5 messy messages.
   - Initializes **BrightPage Stationery Orders** sheet, implements `log_stationery_order`, tests confirmed/unlisted/missing-date queries, and verifies only Ananya's valid order (`12` spiral notebooks, `Rs 102` discount, `Rs 0` delivery, `Rs 918` total) is logged.
6. **Section 6 · Break It & Restore It:** Demonstrates `tool1` (`"does stuff"`) failing to trigger tool calls (`[]`), then restores `get_price` and verifies tool invocation (`['get_price']`).

---

## 🚀 How to Run

### In Google Colab
1. Open [`DAY4/task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY4/task.ipynb) via the **Open In Colab** badge above.
2. Add **`GROQ_API_KEY`** in Colab **🔑 Secrets** and toggle **Notebook access** to **ON**.
3. Run all cells (`Ctrl + F9`) and complete the Google OAuth prompt when `auth.authenticate_user()` runs to create/open the Google Sheets.

### Locally
1. Ensure `GROQ_API_KEY` is set in the root [`.env`](file:///c:/Users/ramuv/AgenticAI/.env) file.
2. Install dependencies: `pip install langchain langchain-groq gspread pydantic python-dotenv`.
3. Open `DAY4/task.ipynb` and run all cells — orders will be persisted locally to [`sweet_crumbs_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY4/sweet_crumbs_orders.csv) and [`brightpage_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY4/brightpage_orders.csv).
