# 🚀 Day 5 – PDF Question Answering using RAG

## 🎯 Overview

This notebook builds a **PDF Question-Answering system** using Retrieval-Augmented Generation (RAG). It extracts text from an uploaded PDF, splits the text into smaller chunks, converts the chunks into embeddings, stores them in a FAISS index, and retrieves relevant content to answer questions with the Groq LLM.

The assistant is instructed to answer only from the PDF context and to say when the answer is not available in the document.

## 🧠 What You Will Learn

- Extract text from PDF files, including scanned pages using OCR.
- Split long text into smaller overlapping chunks.
- Generate text embeddings with Sentence Transformers.
- Store and search embeddings with FAISS.
- Retrieve relevant PDF sections for a question.
- Generate grounded answers using the Groq API.

## 🧰 Technologies Used

| Tool / Library | Purpose |
|---|---|
| PyMuPDF (`fitz`) | Read PDF text and render pages as images |
| Tesseract + `pytesseract` | OCR for scanned PDF pages |
| Pillow | Handle page images |
| Sentence Transformers | Convert text and questions into embeddings |
| `all-MiniLM-L6-v2` | Embedding model used by the RAG pipeline |
| FAISS | Store and search document vectors efficiently |
| NumPy | Work with embedding arrays |
| Groq | Generate answers using `openai/gpt-oss-20b` |

## ⚙️ How the Notebook Works

### 1. Install dependencies

The notebook installs the Python libraries and system tools needed for PDF processing, OCR, embeddings, vector search, and Groq API access.

### 2. Compare embedding models

It loads `all-MiniLM-L6-v2` and `all-mpnet-base-v2`, encodes the sample sentence, and prints each vector's shape. This demonstrates that different embedding models produce vectors with different dimensions.

### 3. Upload and extract PDF text

- Upload a PDF using Google Colab's file upload feature.
- `read_pdf()` checks each page for selectable text.
- If a page has no extractable text, it renders the page as an image and uses Tesseract OCR to read it.
- The extracted page text is combined into `pdf_text`.

**Important:** OCR is used as a fallback for pages where normal text extraction returns no text.

### 4. Split text into chunks

`split_text(text, chunk_size=500, overlap=100)` divides the extracted text into chunks of 500 characters, with 100 characters of overlap between consecutive chunks.

**Why overlap?** It helps preserve context when a useful passage crosses a chunk boundary.

The function also checks that the overlap is smaller than the chunk size.

### 5. Create embeddings and a FAISS index

- `SentenceTransformer("all-MiniLM-L6-v2")` converts each chunk into a numerical vector.
- Embeddings are normalized and converted to NumPy `float32` arrays.
- `faiss.IndexFlatIP` creates an inner-product search index.
- The chunk embeddings are added to the index.

Because the vectors are normalized, inner-product search can be used to find semantically similar text.

### 6. Retrieve relevant PDF context

`retrieve_context(question, top_k=3)`:

1. Converts the question into an embedding using the same model.
2. Searches the FAISS index for the closest matching chunks.
3. Selects up to three relevant chunks.
4. Combines them into the context sent to the language model.

### 7. Generate an answer with Groq

`ask_pdf(question)` sends the retrieved context and the user's question to the Groq chat completion API using `openai/gpt-oss-20b`.

The system instructions tell the model to:

- Answer only from the supplied PDF context.
- Avoid inventing information.
- State **“The answer is not available in the PDF.”** if the context does not contain the answer.
- Keep the response clear and concise.

The API key is requested securely with `getpass()` rather than being written directly in the notebook.

### 8. Ask questions interactively

The final loop accepts questions one at a time. Type `exit` to end the session. For each question, the notebook retrieves matching PDF content and prints the generated answer.

## 🔄 Complete RAG Workflow

```text
📄 Upload PDF
      ↓
📖 Extract text (PyMuPDF / OCR)
      ↓
✂️ Split text into overlapping chunks
      ↓
🧮 Generate embeddings
      ↓
🗂️ Store vectors in FAISS
      ↓
❓ Enter a question
      ↓
🔎 Retrieve the most relevant chunks
      ↓
🤖 Groq LLM generates an answer from the context
      ↓
✅ Display the answer
```

## ▶️ How to Run

1. Open the notebook in **Google Colab**.
2. Run the installation cells.
3. Run the embedding-model demonstration cell.
4. Run the PDF upload and extraction cell, then select your PDF.
5. Run the chunking cell.
6. Run the embedding and FAISS indexing cell.
7. Run the Groq setup and RAG function cell.
8. Enter questions in the final input loop.
9. Type `exit` when you finish.

## 📝 Important Notes

- The notebook uses Google Colab-specific file upload functionality (`google.colab.files`).
- A valid Groq API key is required. Keep it private and do not commit it to GitHub.
- The PDF should contain readable English text for the configured `eng` OCR language.
- The current retrieval setting is `top_k=3`, so up to three chunks are supplied to the model.
- The FAISS index and conversation are created in the running notebook session; this notebook does not save the index for reuse after the session ends.
- Answers depend on the quality of PDF text extraction, OCR, chunking, and retrieval. Always verify important answers against the original PDF.

## ✅ Key Takeaways

- **Embeddings** represent text as numerical vectors so similar meanings can be searched.
- **FAISS** quickly finds document chunks related to a question.
- **RAG** gives the language model relevant document context before it answers.
- **OCR** makes scanned PDF pages usable when direct text extraction is unavailable.
- Grounding the response in retrieved PDF content helps reduce unsupported answers, but does not guarantee perfect accuracy.

---

**🚀 Day 5 Outcome:** A basic PDF-based RAG assistant that can extract document content, retrieve relevant passages, and answer questions using the Groq LLM.
