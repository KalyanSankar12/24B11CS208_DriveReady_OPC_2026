# Day 1 – Building a Bakery AI Assistant with Groq

## Overview
This notebook introduces an LLM-powered customer-support assistant for **Sweet Crumbs Home Bakery**. It uses Groq-hosted language models through LangChain's `ChatGroq` integration and demonstrates how a system prompt can guide an assistant's role, tone, business knowledge, and response limits.

## Learning objectives
- Connect a Python notebook to the Groq API.
- Load an API key securely from Google Colab Secrets.
- Initialize a chat model with a chosen model name and temperature.
- Separate system instructions from the customer's message.
- Use `llm.invoke()` to send messages and receive a response.
- Apply basic prompt-injection detection using a prompt-guard model.

## Tools and libraries
| Tool / library | Purpose |
|---|---|
| Python | Main programming language |
| `langchain-groq` | LangChain integration for Groq chat models |
| `groq` | Groq Python SDK for direct API calls |
| Google Colab `userdata` | Reads the API key from Colab Secrets |
| `os` | Sets the API key as an environment variable |
| `openai/gpt-oss-120b` | Chat model used by the notebook |
| `meta-llama/llama-prompt-guard-2-22m` | Prompt-guard model used in the security example |

## 1. Install and import the dependency
```python
!pip install langchain_groq
from langchain_groq import ChatGroq
import os
from google.colab import userdata
```
The package provides the `ChatGroq` class. `userdata` allows the notebook to retrieve a secret without writing the key directly into the notebook.

## 2. Configure the API key
```python
try:
    os.environ["GROQ_API_KEY"] = userdata.get("GROQ_API_KEY")
except Exception:
    print("Please add GROQ_API_KEY to your Colab Secrets.")
```
In Google Colab, add a secret named `GROQ_API_KEY` and grant the notebook access to it. Do not commit API keys to GitHub.

## 3. Initialize the language model
```python
llm = ChatGroq(
    model="openai/gpt-oss-120b",
    temperature=0
)
```
The model name selects the hosted model. A temperature of `0` requests more consistent, less varied responses.

## 4. Define the assistant's role and business rules
The `SYSTEM` prompt describes the bakery and sets important constraints:
- Cakes are baked to order.
- Orders require at least 24 hours' notice.
- Every cake has an eggless version.
- The assistant must not promise delivery times.
- Unknown information should not be invented.
- Replies should be polite and shorter than 60 words.

A system prompt is important because it gives the model context and boundaries before it receives a customer's question.

## 5. Create a reusable question function
```python
def ask(question):
    return llm.invoke([
        ("system", SYSTEM),
        ("human", question)
    ]).content
```
This function sends the system instructions and the user's question together. `.content` extracts the text of the model's reply.

## 6. Try example customer questions
The notebook tests questions such as:
- “Can I get my order tomorrow?”
- “Can I get an eggless cake for my order?”

The expected behavior is to explain the 24-hour notice policy, confirm that eggless cakes are available, and avoid guaranteeing a delivery time.

## 7. Prompt-guard example
The final section calls the Groq client with the `meta-llama/llama-prompt-guard-2-22m` model and supplies an instruction that attempts to override previous directions. The printed numeric result (`0.9988957643508911` in the provided output) is the model's returned score for that example. It illustrates using a guard model as an additional safety check; it is separate from the bakery chat function.

## How to run
1. Open the notebook in Google Colab.
2. Add `GROQ_API_KEY` under **Secrets** and enable notebook access.
3. Run the installation cell.
4. Run the imports, API configuration, and model initialization cells.
5. Define `SYSTEM` and `ask()`, then run the sample questions.
6. Run the prompt-guard example if you want to explore the security check.

## Key takeaways
- LLM behavior can be shaped by clear system instructions.
- Business facts and policies should be explicitly supplied to the assistant.
- A reusable function makes repeated model calls easier.
- API credentials should be stored in secrets, not source code.
- Safety checks can complement prompting, but should not be treated as a guarantee that every unsafe request will be blocked.

## Notes
The notebook's bakery example is a demonstration, not a connected ordering or delivery system. It does not verify real inventory, accept payments, or schedule deliveries.
