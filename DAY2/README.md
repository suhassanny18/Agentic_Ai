# 📚 Day 2: Prompts, Memory & Building an AI Agent

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JAY4IGNITE/agenticAI/blob/main/DAY2/task.ipynb)

Comprehensive guide, concepts, and implementation details for **Day 2** of the Agentic AI Workshop.

---

## 🎯 Task Overview
Build a complete conversational AI agent for **BrightPage Stationery & Office Supplies** from scratch, featuring in-context few-shot intent classification, structured JSON order extraction, conversational sliding-window memory, production error resilience, and an interactive Colab dashboard.

- **Primary Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY2/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies
- **Model Used:** `openai/gpt-oss-120b` via Groq API

---

## 🧠 Core Engineering Principles

### 1. The 4-Pillar System Prompt
Every production agent prompt requires four calibrated pillars:
- **ROLE:** Identity and scope boundary (*"You are BrightPage Assistant, helping customers with school and office stationery..."*).
- **FACTS:** Ground truth inventory, pricing, delivery policies, and discounts:
  - Ballpoint Pen: ₹10 | Gel Pen: ₹25
  - A4 Ruled Notebook (200 pgs): ₹80 | Unruled Notebook (200 pgs): ₹75
  - Spiral Sketchbook (A4): ₹150 | Sticky Notes (3x3 inch, 100 sheets): ₹40
  - Free delivery on orders above ₹500 (standard delivery: 2-3 business days).
  - Bulk discount: 10% off for 10 or more notebooks.
- **RULES:** Operational guardrails:
  - Never promise same-day delivery.
  - If an item is not in FACTS (e.g., scientific calculators), respond: *"I will check with the store manager and update you."*
- **FORMAT:** Concise, polite, under 60 words, in English or Telugu.

### 2. In-Context Learning (Few-Shot Prompting)
Rather than relying on model intuition, we provide labelled input-output examples in the prompt to enforce strict classification into three discrete classes:
- `order`: Customer desires to purchase items.
- `inquiry`: Customer asks about prices, stock, or policy.
- `complaint`: Customer reports an issue, damage, or delay.

### 3. Structured Output Extraction (JSON)
Agents must interface with downstream APIs and databases. We instruct the model to parse natural conversational customer orders into strict JSON schemas:
```json
{
  "items": [
    {"name": "A4 Ruled Notebook", "quantity": 5},
    {"name": "Gel Pen", "quantity": 10}
  ],
  "delivery_address": "Main Road, Kakinada",
  "contact_number": "9876543210"
}
```

### 4. Conversational Memory by Hand (Sliding Window)
Because LLMs are stateless, client code must maintain the message history:
- Each user turn appends `{"role": "user", "content": ...}`.
- Each assistant response appends `{"role": "assistant", "content": ...}`.
- **Sliding-Window Trimming (`MAX_TURNS`):** As the dialogue grows, older turns are sliced off to avoid exceeding the model's context window and incurring unnecessary token fees.

### 5. Production Error Resilience
Network drops and API rate limits (HTTP 429) can crash long-running agent workflows. API calls are wrapped in robust `try / except` blocks catching `groq.RateLimitError` and returning user-friendly recovery messages.

---

## 🏗️ Notebook Implementation Breakdown

### Section 1: Setup & Environment
Installs `groq`, `python-dotenv`, and `ipywidgets`, followed by multi-tier API key resolution.

### Section 2: 4-Pillar System Prompt & Basic Chat Loop
Establishes the BrightPage stationery domain prompt and basic `ask_stationery(question)` query function.

### Section 3: Few-Shot Customer Intent Classifier
Implements `classify_intent(user_message)` returning exact labels:
```text
Customer: "I want to buy 10 gel pens and 2 spiral sketchbooks" -> [order]
Customer: "Do you have sticky notes in stock?"               -> [inquiry]
Customer: "I received a torn sketchbook in my delivery"        -> [complaint]
```

### Section 4: Structured JSON Order Extractor
Implements `extract_order_json(order_text)` with JSON validation to produce structured records ready for database insertion.

### Section 5: Python Foundations for Agents
- Inventory dictionary with item lookup and pricing calculator.
- Python type hints (`str`, `dict`, `list`) and docstrings.
- Custom `@log_call` execution logging decorator that records function call latency and arguments.

### Section 6: Stateful Agent with Sliding-Window Memory
Implements `BrightPageAgent` class maintaining state:
- `memory`: History list of message dictionaries.
- `max_turns`: Maximum conversational turns preserved (sliding window).
- `chat(user_input)`: Multi-turn conversational interface.
- `clear_memory()`: Reset session context.

### Section 7: Interactive Colab Dashboard
Rich `ipywidgets` dashboard featuring:
- Visual chat bubble stream with multi-turn history.
- Policy verification presets (Bulk discount test, Unlisted item test).
- Intent classification runner widget.
- Structured JSON order extraction widget.

### Section 8: Student's 10% Verification Suite
Three rigorous experimental evaluations:
1. **Temperature Variation ($T=0.2$ vs $T=1.2$):** Proves low temperature maintains factual compliance while high temperature generates creative marketing slogans.
2. **Hallucination Boundary Test:** Inquires about an unlisted item (*"Do you sell scientific calculators?"*) to confirm the model adheres to the rule: *"I will check with the store manager..."*.
3. **Memory Window Limit Test:** Tests memory retention by supplying a customer name in Turn 1, asking unrelated queries, and testing whether the agent remembers or forgets based on the `MAX_TURNS` boundary.

---

## 🚀 How to Run

### In Google Colab
1. Click the **Open In Colab** badge above.
2. Add your `GROQ_API_KEY` in Colab **Secrets** with **Notebook access** set to **ON**.
3. Run all cells (`Ctrl + F9`).

### Locally
1. Ensure the root `.env` file contains your `GROQ_API_KEY`.
2. Open `DAY2/task.ipynb` in VS Code or JupyterLab.
3. Run all cells from top to bottom.
