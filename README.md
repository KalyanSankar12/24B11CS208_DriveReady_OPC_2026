# Day 4 -- Agentic AI: Smart Mobile Store Order Assistant

## 1. What is this task?

In Day 4, you build a simple **AI agent for Smart Mobile Store** using
LangChain.

The assistant can: - Answer questions about mobile prices. - Check the
shop's opening hours. - Collect the details needed for a mobile order. -
Ask the customer to confirm the order. - Save a confirmed order in
Google Sheets.

This task introduces **tools and agents**. Instead of only generating
text, the AI agent can choose and call Python functions to perform
specific actions.

------------------------------------------------------------------------

## 2. Technologies used

  Technology     Purpose
  -------------- --------------------------------------
  Python         Main programming language
  Google Colab   Run the notebook
  LangChain      Build the AI agent and connect tools
  Groq           Provide the language model
  `gspread`      Write order details to Google Sheets
  Pydantic       Validate order information

------------------------------------------------------------------------

## 3. Store information used in this project

### Mobile models and prices

  Mobile model               Price
  -------------------- -----------
  iPhone 15              Rs 55,000
  Samsung Galaxy S24     Rs 45,000
  OnePlus 12             Rs 60,000
  Redmi Note 13          Rs 18,000

### Shop opening hours

  Day                Timings
  ------------------ ---------------
  Monday--Saturday   9 am to 8 pm
  Sunday             10 am to 6 pm

**Order notice:** Orders need 24 hours' notice.

**Order details required:** Customer name, mobile model, quantity,
purchase date, and storage variant.

**Storage options in this exercise:** 128GB, 256GB, and 512GB.

> These are the sample store details configured in this learning
> project. Update the code if the actual store's prices, hours, or
> available storage options change.

------------------------------------------------------------------------

## 4. Step-by-step instructions

Open `Agentic_ai_Day4.ipynb` in Google Colab. Run the cells from top to
bottom.

### Step 1 -- Install the required libraries (Cell 1)

``` python
!pip install -q langchain langchain-groq gspread
```

**What it does:** - `langchain` -- creates the agent and manages tool
calls. - `langchain-groq` -- connects LangChain to Groq. - `gspread` --
connects Python to Google Sheets.

Run this cell once whenever the Colab runtime needs the packages.

### Step 2 -- Connect Groq and Google Sheets (Cell 2)

This cell: 1. Reads the `GROQ_API_KEY` secret from Google Colab. 2.
Creates the Groq chat model. 3. Authenticates your Google account. 4.
Connects Python to Google Sheets. 5. Creates a spreadsheet named **Smart
Mobile Store Orders**. 6. Adds the column headings.

The model setup used in this notebook is:

``` python
from langchain_groq import ChatGroq

llm = ChatGroq(
    model="openai/gpt-oss-20b",
    temperature=0.2,
    max_retries=3
)
```

Before running this cell: - Add your Groq API key in Colab Secrets with
the name `GROQ_API_KEY`. - Enable notebook access to that secret. -
Complete Google authentication when prompted.

**Important -- avoid creating duplicate spreadsheets:**

The notebook currently uses:

``` python
sh = gc.create("Smart Mobile Store Orders")
```

Run this line only when creating the spreadsheet for the first time.

If you restart the runtime and the spreadsheet already exists, replace
it with:

``` python
sh = gc.open("Smart Mobile Store Orders")
ws = sh.sheet1
```

Do not append the column headings again when reopening an existing
sheet.

**Expected result:** A message similar to:

``` text
Sheet ready: https://docs.google.com/spreadsheets/...
```

Open the printed link to view the spreadsheet.

### Step 3 -- Define the store data and order structure (Cell 3)

This cell defines:

-   `MENU` -- mobile model prices.
-   `HOURS` -- shop opening hours.
-   `Order` -- the structure of an order using Pydantic.

The `Order` model checks that: - The customer name is text. - The mobile
model is one of the listed models. - Quantity is at least 1. - Purchase
date is provided as text. - Storage is one of `128GB`, `256GB`, or
`512GB`.

**Why is validation useful?**

It helps prevent invalid order information from being saved.

### Step 4 -- Create the tools (Cell 4)

The agent has three tools.

#### Tool 1: `get_price(item, qty)`

Gets the price of a mobile model and calculates the total for the
requested quantity.

Example:

``` python
print(get_price.invoke({"item": "iphone 15", "qty": 2}))
```

Expected output:

``` text
2 x iphone 15 = Rs 110000
```

#### Tool 2: `is_open(day)`

Checks the shop's opening hours.

Example:

``` python
print(is_open.invoke({"day": "sunday"}))
```

Expected output:

``` text
Open 10 am to 6 pm on sunday.
```

#### Tool 3: `log_order(name, item, qty, date, storage)`

Validates the order details, calculates the total price, and appends the
order to Google Sheets.

**Important:** This tool should be called only after all required
details have been collected and the customer has confirmed the order.

### Step 5 -- Create the AI agent (Cell 5)

This cell: 1. Imports `create_agent`. 2. Defines the assistant's role
and rules in `SYSTEM`. 3. Connects the model with the three tools. 4.
Creates a `chat()` function to send messages to the agent and preserve
conversation history.

The agent is instructed to: - Use `get_price` for prices. - Use
`is_open` for opening hours. - Never guess prices or timings. - Ask for
missing order details. - Save an order only after customer
confirmation. - Read back the saved order and total.

Run the sample question:

``` python
print(chat("Are you open on Sunday? How much are 2 iPhone 15 phones?"))
```

The assistant should report the Sunday hours and the total price of two
iPhone 15 phones.

Expected facts: - Sunday: 10 am to 6 pm. - Two iPhone 15 phones: Rs
1,10,000.

### Step 6 -- Start an order (Cell 6)

The notebook's sample order request is:

``` python
print(chat("I'm Ravi. Please order 2 iPhone 15 phones with 128GB storage for 24 Dec."))
```

The assistant has these details: - Name: Ravi - Model: iPhone 15 -
Quantity: 2 - Storage: 128GB - Purchase date: 24 Dec

The assistant should ask for any missing information or ask the customer
to confirm the order.

**Do not assume that the order has been saved just because you sent the
request.**

### Step 7 -- Confirm the order

Only after the assistant has presented the order details and asked for
confirmation, run:

``` python
print(chat("Yes, I confirm the order."))
```

The agent should then call `log_order` and save the order.

If the assistant has not asked for confirmation or the details are
incorrect, do not send a confirmation message. Correct the details
first.

### Step 8 -- Verify the Google Sheet

Open the **Smart Mobile Store Orders** spreadsheet using the link
printed in Step 2.

Check that a new row appears with the order details:

  name   mobile_model     qty date     storage      total
  ------ -------------- ----- -------- --------- --------
  Ravi   iphone 15          2 24 Dec   128GB       110000

The exact date text is stored as the customer provided it. The total is
calculated from the price in `MENU`.

If the row is present, the order-saving workflow worked.

------------------------------------------------------------------------

## 5. More test messages

Try these one at a time:

``` python
print(chat("How much is one Samsung Galaxy S24?"))
```

``` python
print(chat("Are you open on Saturday?"))
```

``` python
print(chat("What is the price of 3 Redmi Note 13 phones?"))
```

``` python
print(chat("I want to order one OnePlus 12 with 256GB storage."))
```

For the last request, the assistant should ask for missing details such
as the customer's name and purchase date before saving anything.

------------------------------------------------------------------------

## 6. Important concepts learned

  -----------------------------------------------------------------------
  Concept                             Meaning in this project
  ----------------------------------- -----------------------------------
  LLM                                 The language model that understands
                                      and responds to customer messages

  Agent                               The system that decides which tool
                                      to use

  Tool                                A Python function the agent can
                                      call

  Tool calling                        The agent's process of invoking a
                                      function such as `get_price`

  Pydantic model                      Validates the order fields

  Conversation history                Keeps earlier messages available
                                      during the chat

  Google Sheets integration           Stores confirmed orders in a
                                      spreadsheet
  -----------------------------------------------------------------------

### How does the agent work?

``` text
Customer message
       |
       v
   LangChain Agent
       |
       v
 Understand the request
       |
       v
 Choose the required tool
   /       |        \
Price    Opening    Save order
tool     hours tool  tool
   \       |        /
       v
 Tool result
       |
       v
 Agent responds to customer
```

For an order, the saving step should happen only after the customer
confirms.

------------------------------------------------------------------------

## 7. Common problems and solutions

### Problem: `Secret not found: GROQ_API_KEY`

**Solution:** Add `GROQ_API_KEY` under Google Colab's Secrets panel and
enable notebook access.

### Problem: Google authentication does not work

**Solution:** Run the authentication cell again and complete the Google
sign-in/permission steps.

### Problem: A new spreadsheet is created every time

**Solution:** Use `gc.open("Smart Mobile Store Orders")` to open the
existing spreadsheet instead of calling `gc.create(...)` again.

### Problem: `NameError: name 'get_price' is not defined`

**Solution:** Run the cell that defines the tools before running the
agent cell.

### Problem: The order is not appearing in Google Sheets

**Solution:** 1. Check that the correct spreadsheet is open. 2. Check
that all required order details were provided. 3. Check that you
explicitly confirmed the order. 4. Check the Colab output for an error.

### Problem: The agent gives an unexpected response

**Solution:** Read the full response, check the conversation history,
and provide the missing or corrected information. If needed, restart the
runtime and run the notebook cells in order. Be careful not to create a
duplicate spreadsheet.

------------------------------------------------------------------------

## 8. Day 4 completion checklist

-   [ ] Installed the required libraries.
-   [ ] Connected to Groq successfully.
-   [ ] Authenticated Google Sheets.
-   [ ] Created or opened the Smart Mobile Store Orders spreadsheet.
-   [ ] Defined the mobile prices and opening hours.
-   [ ] Created and tested `get_price`.
-   [ ] Created and tested `is_open`.
-   [ ] Created the `log_order` tool.
-   [ ] Created the LangChain agent.
-   [ ] Asked the agent a price and opening-hours question.
-   [ ] Provided all required order details.
-   [ ] Confirmed the order only after reviewing it.
-   [ ] Verified the saved order in Google Sheets.

------------------------------------------------------------------------

## 9. Final outcome

At the end of Day 4, you have a basic **tool-using AI order assistant**
for Smart Mobile Store.

It can answer store questions using Python tools and save confirmed
customer orders in Google Sheets.

**Main learning:** An LLM can understand a customer's request, while an
agent can select and use tools to perform actions.
