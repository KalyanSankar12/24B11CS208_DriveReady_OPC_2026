# Day 2 – Groq Chat Completions, Conversation Memory, and AI Study Buddy

## Overview
This notebook explores calling Groq models through the **OpenAI-compatible Python SDK**. It begins with a simple chat completion, develops a rule-based bakery customer-support chatbot with conversation history and error handling, and then builds an **AI Study Buddy** with three learning modes: Explain, Quiz, and Code.

## Learning objectives
- Connect the OpenAI Python SDK to Groq using a custom `base_url`.
- Send system and user messages to a chat-completion model.
- Design prompts around business facts and response rules.
- Maintain conversation context using a message-history list.
- Add a helper function for price calculation.
- Handle rate-limit and API-connection errors.
- Use a decorator to log function calls.
- Build a chatbot with selectable learning modes.

## Tools and libraries
| Tool / library | Purpose |
|---|---|
| Python | Main programming language |
| `openai` | OpenAI-compatible client used to call Groq |
| Groq API | Hosts the selected language model |
| `google.colab.userdata` | Retrieves `GROQ_API_KEY` securely |
| `json` | Formats menu data in the prompt |
| `functools` | Preserves wrapped function metadata in the logging decorator |
| `RateLimitError`, `APIConnectionError` | Handles common API failures |
| `openai/gpt-oss-20b` | Model used in the notebook |

## 1. Install and configure the client
```python
!pip install -q openai

from google.colab import userdata
from openai import OpenAI

client = OpenAI(
    api_key=userdata.get("GROQ_API_KEY"),
    base_url="https://api.groq.com/openai/v1",
)
```
Although the code imports the OpenAI SDK, the custom base URL sends requests to Groq's OpenAI-compatible endpoint. The API key is read from Colab Secrets.

## 2. Make a basic chat-completion request
The first request uses `client.chat.completions.create()` with:
- `model="openai/gpt-oss-20b"`
- A system message describing the assistant as a funny joker.
- A user message asking for a joke.

The response text is read from `response.choices[0].message.content`.

## 3. Create the Sweet Crumbs bakery assistant
The next prompt defines the bakery's menu and policies:
- Plum cake – ₹500
- Chocolate cake – ₹650
- Cupcake – ₹80
- Orders need 24 hours' notice.
- All cakes have eggless versions.
- The assistant must not promise delivery time or place.
- Unknown facts should be answered with “I'll check with the owner.”
- Replies should be friendly, under 50 words, and match English or Telugu used by the customer.

The model is called with a low temperature (`0.2`) to encourage consistent customer-service answers. `response.usage` is also printed to inspect token usage.

## 4. Add helper functions, logging, and conversation history
The notebook then introduces several Python concepts.

### Menu and price helper
`MENU` stores item prices. `get_price(item, qty=1)` looks up an item's price and calculates the total for the requested quantity.

### Logging decorator
`log_call` is a decorator. It runs the original function, prints the function name, arguments, and result, and returns the result. `@log_call` applies it to `get_price`.

### Conversation memory
`history` stores the system prompt and subsequent user/assistant messages. The `chat()` function appends the new user message, sends a limited recent context (`MAX_TURNS`), calls the model, and appends the reply. This allows follow-up questions to use earlier conversation context.

### Error handling
- `RateLimitError`: returns a friendly message asking the customer to try again.
- `APIConnectionError`: returns a message explaining that the server cannot currently be reached.

The interactive loop accepts customer messages until the user types `bye`, `exit`, or `quit`.

## 5. AI Study Buddy – project concept
The notebook next describes an AI-powered study assistant for Computer Science students. Its intended subjects include:
- Data Structures and Algorithms (DSA)
- Python
- C++
- Java
- General Computer Science concepts

Its three modes are:
| Mode | Behavior |
|---|---|
| `explain` | Explains concepts from the basics with examples |
| `quiz` | Asks one question at a time and evaluates the student's answer |
| `code` | Focuses on programming examples, code, and dry runs |

## 6. Implement the Study Buddy
The Study Buddy system prompt defines the assistant's role and teaching rules: use simple English, include examples, explain code step by step, provide beginner-friendly coding solutions, ask practice questions in quiz mode, and admit uncertainty.

`get_mode_instruction(mode)` returns the instruction for a selected mode. If an unknown mode is supplied, it falls back to Explain Mode.

The `chat(user_text, mode)` function:
1. Adds the user's message to conversation history.
2. Builds the model messages with the main system prompt and current mode instruction.
3. Sends the request to Groq.
4. Handles rate-limit and connection errors.
5. Stores and returns the assistant's reply.

The interactive loop supports commands such as `/mode quiz` and `/mode code`. Type `bye` to end the session.

## How to run
1. Open the notebook in Google Colab.
2. Add a Colab Secret named `GROQ_API_KEY` and enable access.
3. Run the installation and client-setup cells.
4. Run the basic joke example.
5. Run the bakery assistant sections.
6. Run the Study Buddy implementation.
7. Try each mode using `/mode explain`, `/mode quiz`, and `/mode code`.

## Key takeaways
- The OpenAI SDK can communicate with Groq when configured with Groq's compatible endpoint.
- System prompts establish an assistant's role, facts, and response constraints.
- Conversation history provides context for follow-up messages.
- Decorators can add logging without changing a function's main logic.
- Exception handling helps a chatbot respond gracefully to API failures.
- A single chatbot can support different interaction styles through mode-specific instructions.

## Notebook checks before publishing
- Review the `MENU` dictionary in the bakery section. In the supplied notebook text, the cupcake entry appears malformed; it should be `"cupcake": 80` if that is the intended menu item.
- Confirm that `history` and the model client are initialized by running the cells in order.
- Keep the API key in Colab Secrets and never paste it into the notebook or README.
