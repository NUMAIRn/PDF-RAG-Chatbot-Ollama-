# PDF RAG Chatbot (Ollama)

A local, private chatbot that answers questions about a PDF. It uses Retrieval-Augmented Generation (RAG): the PDF is split into chunks, embedded with Ollama, and the most relevant chunks are given to a local LLM to answer your question. Nothing leaves your machine.

## Features

- Runs fully offline with [Ollama](https://ollama.com)
- Works on a regular laptop (CPU is fine with small models)
- Shows which PDF pages each answer came from
- Streams answers token by token
- Caches embeddings so a PDF is only processed once
- Remembers the last few exchanges for follow-up questions

## Requirements

- Python 3.9+
- [Ollama](https://ollama.com) installed and running
- A PDF with selectable text (scanned PDFs need OCR first)

## Setup

1. **Install Ollama** from https://ollama.com and make sure it is running.

2. **Pull the models:**

   ```bash
   ollama pull llama3.2
   ollama pull nomic-embed-text
   ```

3. **Install the Python packages:**

   ```bash
   python -m pip install ollama pypdf numpy
   ```

## Usage

Pass the path to your PDF as an argument:

```bash
python rag_chatbot.py path/to/document.pdf
```

Examples:

```bash
# PDF in the same folder
python rag_chatbot.py mydoc.pdf

# Windows, path with spaces
python rag_chatbot.py "C:\Users\You\Documents\my report.pdf"

# Mac/Linux
python3 rag_chatbot.py ~/Documents/report.pdf
```

Then ask questions at the prompt. Type `exit`, `quit`, or `q` to leave.

```
You: What is the main conclusion of the report?
Bot: The report concludes that ...
     (sources: pages 12, 14)
```

## Configuration

Edit the settings at the top of `rag_chatbot.py`:

| Setting | Default | Description |
|---|---|---|
| `CHAT_MODEL` | `llama3.2` | Ollama model used to generate answers |
| `EMBED_MODEL` | `nomic-embed-text` | Ollama model used for embeddings |
| `CHUNK_SIZE` | `800` | Characters per chunk |
| `CHUNK_OVERLAP` | `150` | Characters shared between neighboring chunks |
| `TOP_K` | `4` | Number of chunks retrieved per question |
| `CACHE_DIR` | `.rag_cache` | Where embeddings are cached |

### Choosing a chat model

| Hardware | Suggested model |
|---|---|
| 4-8 GB RAM, older CPU | `llama3.2:1b` |
| 8-16 GB RAM | `llama3.2:3b` (default) |
| 16 GB+ RAM or 8 GB GPU | `llama3.1:8b` |

Pull the model first (`ollama pull <name>`), then set `CHAT_MODEL` to match. If answers are slow, lower `TOP_K` to 3.

## How it works

1. **Load:** `pypdf` extracts text from each page.
2. **Chunk:** Each page is split into overlapping chunks, keeping page numbers.
3. **Embed:** Chunks are embedded with `nomic-embed-text` and saved to `.rag_cache/`.
4. **Retrieve:** Your question is embedded, and the top `TOP_K` chunks are found by cosine similarity.
5. **Generate:** The chunks and recent chat history are sent to the chat model, which is told to answer only from the provided context.

## Troubleshooting

**`No module named ollama`**
`pip` and `python` point to different installs. Use `python -m pip install ollama` (or `py -m pip` on Windows, `python3 -m pip` on Mac/Linux). Also check that no file in your folder is named `ollama.py`, and that your IDE uses the same interpreter.

**`Connection refused` or cannot reach Ollama**
The Ollama app isn't running. Start it, or run `ollama serve` in a terminal.

**`model not found`**
Pull the model: `ollama pull llama3.2` and `ollama pull nomic-embed-text`.

**`No extractable text found`**
The PDF is probably scanned images. Run OCR first, for example `ocrmypdf input.pdf output.pdf`.

**Answers are wrong or say "not found in the document"**
Try a larger `TOP_K`, a larger chat model, or a different `CHUNK_SIZE`. If you change `EMBED_MODEL` or the chunk settings, delete `.rag_cache/` so the PDF is re-indexed.

**Slow responses**
Use a smaller model (`llama3.2:1b`), lower `TOP_K`, or close other heavy applications.

## Limitations

- Only text is read. Images, charts, and scanned pages are ignored.
- Tables may be extracted with messy formatting.
- One PDF at a time.
- Answers are only as good as the retrieved chunks and the size of the model used.

## License

Use and modify freely.
