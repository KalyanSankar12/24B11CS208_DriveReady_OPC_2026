# 🚀 Day 6 – Building a Multi-Agent Smart Mobile Store Assistant with LangGraph

## 📌 Overview

In Day 6, we build a **Smart Mobile Store AI assistant** using **LangChain, LangGraph, and Groq**. The system classifies each customer message and routes it to the right handler:

- 🛒 **Order** – handles mobile phone purchase requests.
- ❓ **FAQ** – answers questions about prices, store timings, and policies.
- 👋 **Other** – handles greetings or unrelated messages.

The project uses separate FAQ and Order agents connected through a LangGraph workflow.

## 🎯 Learning Objectives

- Configure a Groq model in Google Colab.
- Use Pydantic for structured output.
- Classify customer messages into intents.
- Create specialized LangChain agents.
- Build conditional routing with LangGraph.
- Visualize and stream graph execution.
- Test customer queries and order logging.

## 🧰 Technologies Used

| Technology | Purpose |
|---|---|
| Python | Main programming language |
| Google Colab | Notebook execution environment |
| Groq | Language model provider |
| LangChain | Model integration, structured output, and agents |
| LangGraph | Graph workflow and conditional routing |
| Pydantic | Structured intent definition |
| IPython | Displays the workflow diagram |

## ⚙️ Step 1 – Install Packages

```python
!pip install -q langchain langchain-groq langgraph
```

- `langchain` – tools for working with language models and agents.
- `langchain-groq` – connects LangChain to Groq.
- `langgraph` – builds graph-based workflows.

## 🔐 Step 2 – Configure the Groq API Key

```python
import os
from google.colab import userdata
os.environ["GROQ_API_KEY"] = userdata.get("GROQ_API_KEY")
```

In Google Colab, add a secret named `GROQ_API_KEY` and enable notebook access. Never publish your private API key.

## 🤖 Step 3 – Initialize the Groq Model

The notebook first initializes `openai/gpt-oss-120b`, then later reassigns the model to `llama-3.3-70b-specdec` with temperature `0`:

```python
from langchain_groq import ChatGroq
llm = ChatGroq(model="llama-3.3-70b-specdec", temperature=0)
```

`temperature=0` is used for more consistent classification. The later model configuration is used for the Smart Mobile Store agents.

## 🧠 Step 4 – Define the Intent Structure

```python
from typing import Literal
from pydantic import BaseModel, Field

class Intent(BaseModel):
    intent: Literal["order", "faq", "other"] = Field(
        description="order = purchase; faq = prices, timings or policies; other = unrelated"
    )

classifier = llm.with_structured_output(Intent)
```

| Intent | Meaning | Example |
|---|---|---|
| `order` | Customer wants to buy a phone | “I want to order an iPhone 15.” |
| `faq` | Customer asks about price, timing, or policy | “What is the price of iPhone 15?” |
| `other` | Greeting or unrelated message | “Good morning!” |

Structured output makes the model return an intent using the defined Pydantic structure.

## 🗺️ Step 5 – Define the Graph State

```python
from typing_extensions import TypedDict

class BakeryState(TypedDict):
    message: str
    intent: str
    reply: str
```

- `message` – customer's original message.
- `intent` – category identified by the classifier.
- `reply` – response from the selected handler.

The notebook retains the name `BakeryState`, although the later workflow is for Smart Mobile Store.

## 🔍 Step 6 – Classify the Customer Message

The `classify()` function sends the message with Smart Mobile Store classification instructions and returns the detected intent:

```python
def classify(state: BakeryState) -> dict:
    result = classifier.invoke([
        ("system", "Classify messages for Smart Mobile Store..."),
        ("human", state["message"])
    ])
    return {"intent": result.intent}
```

Purchase requests become `order`; questions about prices, timings, and policies become `faq`; greetings and unrelated messages become `other`.

## 🧑‍💼 Step 7 – Create Two Specialized Agents

The notebook creates two agents using LangChain's `create_agent()`.

### ❓ FAQ Agent

Handles questions about mobile prices, store opening hours, policies, and advance notice. Its prompt instructs it to use:

- `get_price` for mobile prices.
- `is_open` for store timings.
- `search_faq` for store policies and advance notice.

It must not invent prices or policies. If information is unavailable, it should say: **“I'll check with the owner.”**

### 🛒 Order Agent

Helps customers order mobile phones. Before saving, it should collect:

- Customer name
- Mobile model
- Quantity
- Storage (128GB, 256GB, or 512GB)
- Order date

It is instructed to use `get_price` for the total and `log_order` only after customer confirmation. Orders require 24 hours' advance notice.

## 🧩 Step 8 – Create Agent Execution Functions

`ask_agent()` sends a message to the selected agent and returns its response. The handler functions connect each route to its agent:

| Function | Responsibility |
|---|---|
| `answer_faq()` | Sends the question to the FAQ agent |
| `take_order()` | Sends the purchase request to the Order agent |
| `other()` | Returns a welcome message |

This separation keeps each handler focused on one task.

## 🔀 Step 9 – Build the LangGraph Workflow

Create the graph and register its nodes:

```python
from langgraph.graph import StateGraph, START, END

builder = StateGraph(BakeryState)
builder.add_node("classify", classify)
builder.add_node("faq", answer_faq)
builder.add_node("order", take_order)
builder.add_node("other", other)
builder.add_edge(START, "classify")
```

The routing function reads the intent:

```python
def route(state: BakeryState) -> str:
    return state["intent"]
```

Conditional edges choose the correct handler:

```python
builder.add_conditional_edges(
    "classify", route,
    {"order": "order", "faq": "faq", "other": "other"}
)

builder.add_edge("faq", END)
builder.add_edge("order", END)
builder.add_edge("other", END)
graph = builder.compile()
```

### 🔁 Workflow

```text
             Customer Message
                    |
                    v
              Classify Intent
                    |
          +---------+---------+
          |         |         |
          v         v         v
        ORDER      FAQ      OTHER
          |         |         |
          v         v         v
      Order Agent FAQ Agent Welcome Reply
          |         |         |
          +---------+---------+
                    |
                    v
                   END
```

## 🖼️ Step 10 – Visualize the Graph

```python
from IPython.display import Image, display
display(Image(graph.get_graph().draw_mermaid_png()))
```

This displays the graph so you can see the nodes and paths used by the workflow.

## 🧪 Step 11 – Test the Workflow

The notebook tests messages such as:

```python
tests = [
    "What is the price of iPhone 15?",
    "I am Ravi. I want to order 2 iPhone 15 phones with 128GB storage for 24 December.",
    "Good morning!",
    "What are your store timings?",
    "Do orders require advance notice?"
]
```

It uses `graph.stream(..., stream_mode="updates")` to print execution updates as the graph runs.

| Test message | Expected route |
|---|---|
| Phone price question | FAQ |
| Phone purchase request | Order |
| Greeting | Other |
| Store timings question | FAQ |
| Advance notice question | FAQ |

The actual response depends on the model and the tools available at runtime.

## 💾 Step 12 – Test Order Logging

The final cell invokes `log_order` with a sample order for Ravi:

```python
result = log_order.invoke({
    "name": "Ravi",
    "mobile_model": "iPhone 15",
    "qty": 2,
    "date": "2026-12-24",
    "storage": "128GB",
    "total": 110000
})
```

The notebook prints `orders` before and after the call to check whether the order was added to the collection.

## ▶️ How to Run the Notebook

1. Open the Day-6 `.ipynb` notebook in Google Colab.
2. Install the required packages.
3. Add `GROQ_API_KEY` in Colab Secrets and enable access.
4. Run the Groq model initialization cell.
5. Execute the intent model and graph-state definitions.
6. Ensure `search_faq`, `get_price`, `is_open`, and `log_order` are defined or imported.
7. Create the FAQ and Order agents.
8. Define the handler functions.
9. Build and compile the LangGraph workflow.
10. Display the graph diagram.
11. Run the customer-message tests.
12. Run the order-logging test after defining `orders` and the expected tool schema.

## 📚 Important Concepts

| Concept | Meaning |
|---|---|
| Structured output | Model response follows a defined structure |
| Pydantic model | Defines allowed intent values |
| Intent classification | Identifies what the customer wants |
| LangChain agent | Uses a model and tools to perform a task |
| System prompt | Defines an agent's role and instructions |
| Graph state | Carries data between graph nodes |
| Graph node | Performs one workflow step |
| Conditional edge | Selects the next node based on state |
| Streaming | Displays execution updates as they occur |
| Tool calling | Lets an agent use application functions |

## ⚠️ Notes to Remember

- The beginning of the supplied notebook contains an earlier bakery-intent example. Later sections adapt the classifier and graph for **Smart Mobile Store**.
- The first FAQ graph uses a placeholder reply (`"…answer from search_faq…"`). The later section replaces the FAQ handler with an FAQ agent.
- The pasted code refers to `search_faq`, `get_price`, `is_open`, `log_order`, and `orders`, but their full definitions are not included in the supplied code. Ensure these exist in earlier notebook cells or imports before running dependent cells.
- The final order test assumes `log_order` accepts the fields shown and that `orders` has been initialized.
- The Order agent prompt asks for customer confirmation before saving. A prompt alone is not a strict application-level safeguard; production code should enforce confirmation before invoking the save tool.
- The model is first set to `openai/gpt-oss-120b`, then reassigned to `llama-3.3-70b-specdec`. The later configuration is used for the mobile-store agent setup.

## ✅ Key Takeaways

By completing Day 6, you learned how to classify customer messages, use Pydantic structured output, create separate FAQ and Order agents, route requests with LangGraph conditional edges, visualize and stream graph execution, and test an order-logging tool.

## 🏁 Final Summary

**Day 6 demonstrates a multi-agent customer-support workflow.** A classifier identifies the customer's intent, LangGraph routes the message, and a specialized agent handles the request. This structure makes the Smart Mobile Store assistant organized and easier to extend.

🚀 **End of Day 6 – Multi-Agent AI Workflow with LangChain and LangGraph**
