# Day 3 – LangChain Prompt Templates, Chains, Agents, and Testing

## Overview
This notebook moves from direct SDK calls toward **LangChain** abstractions. It configures a Groq-hosted model through `ChatOpenAI`, creates a prompt-template chain with conversation history, tests responses using CSV test cases, introduces a LangChain agent, and adapts the assistant scenario from a bakery to a **Smart Mobile Store**.

The mobile-store section uses fixed product prices and demonstrates prompt refinement and temperature changes, followed by a small comparison of tasks suited to plain code, a single LLM call, or an agent.

## Learning objectives
- Configure a Groq model through LangChain's OpenAI-compatible chat interface.
- Build reusable prompts using `ChatPromptTemplate`.
- Insert conversation history with `MessagesPlaceholder`.
- Parse model output into plain text using `StrOutputParser`.
- Compose a chain with the pipe (`|`) operator.
- Preserve context across multiple questions.
- Evaluate answers against CSV-based expected keywords.
- Create a LangChain agent and invoke it.
- Improve a system prompt and compare test results.
- Understand the role of temperature in response variation.

## Tools and libraries
| Tool / library | Purpose |
|---|---|
| Python | Main programming language |
| `langchain` | LangChain framework and agent creation |
| `langchain-openai` | OpenAI-compatible chat model integration |
| `ChatOpenAI` | Connects LangChain to the Groq endpoint |
| `ChatPromptTemplate` | Builds structured chat prompts |
| `MessagesPlaceholder` | Inserts prior conversation messages |
| `StrOutputParser` | Converts model output to a string |
| `csv` | Reads test cases from a CSV file |
| Groq API | Hosts `openai/gpt-oss-20b` |

## 1. Configure LangChain with Groq
```python
from google.colab import userdata
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="openai/gpt-oss-20b",
    api_key=userdata.get("GROQ_API_KEY"),
    base_url="https://api.groq.com/openai/v1",
    temperature=0.3
)
```
`ChatOpenAI` is used because Groq provides an OpenAI-compatible API endpoint. The API key is retrieved from Colab Secrets.

## 2. Build a prompt template and chain
The bakery assistant's prompt includes menu prices, the 24-hour notice policy, eggless-cake availability, and rules against inventing delivery information.

```python
prompt = ChatPromptTemplate.from_messages([
    ("system", SYSTEM),
    MessagesPlaceholder("history"),
    ("human", "{question}"),
])

chain = prompt | llm | StrOutputParser()
```
The components work in sequence:
1. `ChatPromptTemplate` creates the messages.
2. `MessagesPlaceholder("history")` inserts previous turns.
3. `llm` generates the response.
4. `StrOutputParser` returns the answer as text.

The `ask()` function invokes the chain, then adds the question and answer to `history`. This allows a follow-up such as asking the assistant to recall a name introduced earlier.

## 3. Test the assistant with CSV data
`run_tests()` reads a CSV file with columns named `question` and `must_include`. For each row, it sends the question to the chain and checks whether the expected phrase appears in the answer. It prints `PASS` or `FAIL` and a final pass count and percentage.

This is a simple keyword-based check. It does not fully judge factual correctness, tone, or semantic similarity, but it provides a repeatable first evaluation.

## 4. Introduce a LangChain agent
The notebook uses `create_agent()` with the configured model, an empty tools list, and the bakery system prompt. It invokes the agent with a user message and prints the final message content.

This demonstrates the agent interface. Since `tools=[]`, this particular example does not give the agent external tools to call.

## 5. Change the use case to a Smart Mobile Store
The notebook then selects a mobile-shop assistant and supplies these product facts:

| Product | Price |
|---|---:|
| iPhone 15 | ₹55,000 |
| Samsung Galaxy S24 | ₹45,000 |
| OnePlus 12 | ₹60,000 |
| Redmi Note 13 | ₹18,000 |

The system prompt instructs the assistant to:
- Use only the supplied product facts.
- Never invent prices or product details.
- Say “I'll check with the owner” when information is unavailable.
- Avoid promising delivery times or places.
- Calculate totals carefully.
- Reply in fewer than 50 words and match the customer's English or Telugu.

The notebook asks for product prices and availability, then reads `tests.csv` and runs the test questions against the mobile-store chain.

## 6. Improve the system prompt and compare tests
The notebook updates the system prompt with more explicit instructions, including:
- Always use the supplied facts.
- State the exact listed price when asked for a product price.
- Avoid invented details.
- Use the fallback response for unavailable information.
- Do not promise delivery time or place.
- Calculate totals carefully.

It rebuilds the chain and runs the CSV tests again. This illustrates prompt iteration: make the requirements more specific, then evaluate whether the responses satisfy the expected checks.

## 7. Explore temperature
The notebook changes the model temperature from `0.3` to `1.0`, rebuilds the chain, and runs the tests twice.

Temperature controls response variability. A lower value generally produces more consistent outputs, while a higher value can produce more varied wording. Repeated test runs may therefore differ. For factual customer-support answers, consistency is usually important.

## 8. Decide whether a task needs code, an LLM call, or an agent
The final section classifies three example tasks:

| Example task | Approach shown |
|---|---|
| Calculate the total price of 3 mobile phones | Plain Code |
| Answer customer questions about mobile features | One LLM Call |
| Check mobile stock availability using a tool | Agent |

The idea is to choose the simplest suitable approach. Exact arithmetic is usually better handled by deterministic code; natural-language responses can use an LLM; and tasks that need the model to select and use external tools are a stronger fit for an agent.

## How to run
1. Open the notebook in Google Colab.
2. Install `langchain` and `langchain-openai`.
3. Add `GROQ_API_KEY` to Colab Secrets and enable access.
4. Run the model setup and prompt-template cells.
5. Run the memory example.
6. Make sure the required CSV file is available in the notebook's working directory.
7. Run the CSV tests, agent example, and mobile-store sections in order.
8. Compare the test results before and after prompt refinement, then try the temperature experiment.

## CSV test-file format
The test file should contain the two columns used by the notebook:

```csv
question,must_include
What is the price of iPhone 15?,55000
Do you have Redmi Note 13?,Redmi Note 13
```
These are illustrative rows; use the actual questions and expected phrases appropriate to your test cases.

## Key takeaways
- LangChain separates prompt construction, model invocation, and output parsing into reusable components.
- Message history enables conversational context.
- CSV-driven checks make prompt evaluation repeatable.
- Prompt clarity can improve adherence to business rules.
- Temperature affects variation, not factual reliability.
- Agents are useful when a task requires tool selection or external actions; an agent with no tools has no external capability.

## Notebook checks before publishing
- The first test function defaults to `/content/test.csv`, while the later mobile-store test uses `tests.csv`. Make sure the intended file exists at the path used by the cell.
- The notebook prints the contents of `tests.csv`; ensure that file is included in the repository if readers need to reproduce the tests.
- The sample CSV above is only a format illustration, not a claim about the exact contents of your original test file.
- The agent example uses an empty tools list, so do not describe it as checking live stock.
- Keep `GROQ_API_KEY` in Colab Secrets. Never commit credentials.
