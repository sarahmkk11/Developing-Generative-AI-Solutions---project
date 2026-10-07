# Developing-Generative-AI-Solutions---project
# SmartAssistant: RAG Pipeline with Telegram Integration

SmartAssistant is a notebook-based study assistant that retrieves information from three text documents and uses Google Gemini to generate answers from the retrieved context. A Telegram bot provides an interface for questions, document search, summaries, and practice quizzes.

The project implements **Option 1: Foundational Data Pipeline for Retrieval-Augmented Generation (RAG)**. Telegram is an additional interface to the same pipeline.

## Training Program

This project was developed as part of the **Generative AI Solutions Development Program (تطوير حلول الذكاء الاصطناعي التوليدي)** at **SDAIA Academy**.

- **Academy GitHub:** [SDAIAAcademy](https://github.com/SDAIAAcademy)
- **Project option:** Foundational Data Pipeline for Retrieval-Augmented Generation (RAG)

## Project Workflow

1. **Ingestion:** Load three TXT files, split them into dynamic chunks, generate embeddings, and save a FAISS index with metadata.
2. **Retrieval:** Load the saved database, embed the user question with the same model, and retrieve the top three chunks using cosine similarity.
3. **Generation:** Send the question and retrieved chunks to Gemini, then display the generated answer and the source filenames associated with its cited chunks.

Ingestion is skipped when a complete saved database exists. Query embeddings are still generated for each new question.

## Features

- Three input documents from different domains.
- Dynamic chunking that respects paragraph and sentence boundaries, with word-based splitting for unusually long sentences.
- Multilingual SentenceTransformers embeddings.
- FAISS `IndexFlatIP` with normalized vectors for cosine similarity.
- Persistent index, chunk metadata, and configuration manifest.
- Top-three retrieval with text, similarity score, source filename, and chunk number.
- Gemini answers instructed to use only the retrieved context.
- Validation of generated reference IDs against retrieved chunks.
- Telegram questions, search, summaries, quizzes, and local answer grading.
- Separate conversation state for each Telegram user.

## Requirements

- Google Colab, recommended for running the notebook.
- A Google account for persistent storage on Google Drive.
- A valid Gemini API key and access to the configured model.
- A Telegram bot token for the optional Telegram interface.
- Three nonempty UTF-8 TXT files from different domains.
- Internet access to install packages, download the embedding model, and connect to Gemini and Telegram.

The notebook installs `google-genai`, `sentence-transformers`, `faiss-cpu`, and `python-telegram-bot`.

## Setup and Execution

### 1. Open the notebook

Upload `SmartAssistant.ipynb` to Google Colab and run the cells in order.

### 2. Configure secrets

Add the following entries in Colab's **Secrets** panel and enable notebook access:

| Secret | Purpose |
|---|---|
| `Gemini_Project` or `GEMINI_API_KEY` | Gemini API authentication |
| `TELEGRAM_BOT_TOKEN` | Telegram bot authentication |

The notebook also accepts environment variables or prompts for credentials using hidden input. Do not hardcode credentials in the notebook.

### 3. Mount Google Drive

Keep `USE_GOOGLE_DRIVE = True` and authorize the Drive mount. By default, files are stored under:

```text
/content/drive/MyDrive/SmartAssistant_RAG/
```

For local Jupyter use, set `USE_GOOGLE_DRIVE = False` and place your three TXT files in the printed `documents` directory. The Google Colab upload interface is unavailable outside Colab.

### 4. Provide three documents

On the first run, upload exactly three `.txt` files together. Example domains:

- World foods
- Artificial intelligence
- Data warehousing

The code checks the file count and content but does not automatically verify that the domains differ. Choose documents with enough content to produce at least three chunks in total.

Alternatively, set `USE_DEMO_FILES = True` to create the built-in sample documents when the documents directory is empty.

### 5. Build or load the database

Run Phase 1. On the first run, the notebook creates the index and metadata. Subsequent runs load them and display:

```text
Phase 1 skipped: loading pre-computed database.
```

### 6. Test retrieval

Phase 2 runs an example question and prints the top three retrieved chunks. Adjust `EXAMPLE_QUESTION` to match your documents.

### 7. Generate an answer

The generation example is commented out by default. Uncomment and run:

```python
response = ask(EXAMPLE_QUESTION)
print_retrieval(response['question'], response['retrieved'])
print('\n' + format_answer(response))
```

To ask multiple questions interactively, uncomment:

```python
interactive_search()
```

Enter `q` to exit.

### 8. Start Telegram

Run the Telegram definitions cell, then uncomment and execute:

```python
await start_bot()
```

Open your bot in Telegram and send `/start`. Keep the Colab session connected while using the bot. To stop it, execute:

```python
await stop_bot()
```

## Telegram Commands

| Command | Behavior |
|---|---|
| `/start` or `/help` | Display available commands. |
| `/ask <question>` | Generate a RAG answer with source references. Ordinary text messages do the same. |
| `/search <question>` | Return the top three chunks and similarity scores without a Gemini call. |
| `/summarize <text>` | Summarize the supplied text. Without text, summarize the user's last retrieved chunks. |
| `/quiz <topic>` | Retrieve context and generate two multiple-choice questions. Without a topic, use the user's last retrieved chunks. |
| `/answer 1 2` | Grade the two selected answers, reveal the correct options, and show explanations and sources. |

Quiz options are numbered from 1 to 4. The answer key is not shown until grading. User state is held in memory and is lost when the bot is restarted.

## Configuration

| Setting | Default | Purpose |
|---|---|---|
| `USE_GOOGLE_DRIVE` | `True` | Persist project files across Colab sessions. |
| `EMBEDDING_MODEL` | `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` | Embed documents and questions using the same model. |
| `GEMINI_MODEL` | `gemini-2.5-flash` | Generation model; configurable through the `GEMINI_MODEL` environment variable. Availability depends on the API account. |
| `CHUNK_WORDS` | `100` | Word budget for dynamic chunks. |
| `FORCE_REBUILD` | `False` | Rebuild the database deliberately. |
| `USE_DEMO_FILES` | `False` | Use built-in samples when no TXT files are present. |

## Saved Files

| File or folder | Contents |
|---|---|
| `documents/` | The three source TXT files. |
| `vector_db/vector_db.index` | Saved FAISS vectors. |
| `vector_db/metadata.json` | Chunk text, source filename, and chunk number. |
| `vector_db/manifest.json` | Embedding settings, dimensions, document hashes, and database integrity hashes. |

To update the corpus, replace the files in `documents/`, ensure exactly three TXT files remain, set `FORCE_REBUILD = True`, and rerun the ingestion cells. Set it back to `False` afterward. If the directory already contains documents, the upload cell does not prompt for replacements.

## Understanding the Output

**Retrieval output** contains the user question, the three retrieved passages, their cosine scores, and their filenames.

**Generation output** contains the question, Gemini's answer, and the filenames mapped from the reference IDs it reports using. Inline citations such as `[R1]` identify retrieved chunks.

Retrieval sources and references used by the LLM can differ: a generated answer may use only one or two of the three retrieved chunks. For insufficient context, the model is instructed to return an explanation with no references.

Cosine scores are similarity measurements, not confidence percentages or probabilities that an answer is correct.

## Troubleshooting

| Problem | What to check |
|---|---|
| No generated answer or LLM references appear | `search_rag()` only retrieves chunks. Run `ask()` and `format_answer()`, or use Telegram `/ask`. The notebook generation example is commented out by default. |
| `Model returned inconsistent references` | The model's inline citations and reported reference IDs disagree. Retry the question; if persistent, inspect the generation response and model configuration. The notebook rejects inconsistent references rather than displaying them. |
| `None — insufficient context` appears | The generated response reported no used sources. Check whether the documents contain the requested information and inspect the retrieved chunks. |
| Gemini request fails | Check the secret, notebook access, model availability, connection, and API quota. |
| Database settings differ or a source changed | Restore the previous settings/files, or deliberately rebuild with `FORCE_REBUILD = True`. |
| Database is incomplete or corrupt | Set `FORCE_REBUILD = True` and rerun ingestion using the original documents. |
| The upload is rejected | Upload exactly three UTF-8 `.txt` files together. |
| The embedding model cannot load | Check internet access and rerun the setup cell. |
| Telegram does not respond | Run `await start_bot()`, check the token, and keep the runtime connected. Stop other active instances of the same bot. |
| `/quiz` or `/summarize` without arguments does not work | Search or ask a question first, or provide a topic/text directly. |

## Validation Status and Limitations

The notebook was checked for Python syntax and exercised with mocked embeddings and FAISS for chunking, persistence, top-three retrieval, and changed-source detection. Live Gemini and Telegram execution was not verified with user credentials.

- Grounding instructions and reference validation reduce unsupported output but do not prove factual correctness.
- Reference validation checks IDs and citation consistency; it does not independently verify every claim.
- Dynamic chunking uses document structure and a word budget; it is not embedding-based semantic boundary detection.
- The most similar three chunks may still be unrelated to the question.
- Summary sources identify the supplied context, rather than claim-by-claim verified citations.
- Gemini API calls may be subject to account limits or charges.
- The Telegram bot runs inside the active notebook session; this is not an always-on deployment.

## Documentation

- [Google Gen AI Python SDK](https://googleapis.github.io/python-genai/index.html)
- [python-telegram-bot Application lifecycle](https://docs.python-telegram-bot.org/telegram.ext.application.html)

## Technical Documentation

### Main Components

| Function | Responsibility |
|---|---|
| `dynamic_chunks(text, max_words)` | Group sentences within paragraphs into bounded chunks; split oversized sentences at word boundaries. |
| `encode(texts)` | Generate contiguous float32 embeddings and normalize them. |
| `build_database()` | Validate three source documents, embed chunks, and save the index, metadata, and manifest. |
| `load_database()` | Check saved settings, integrity hashes, and source changes before loading the database. |
| `search_rag(question, top_k=3)` | Embed a question and return ranked chunks with scores and reference IDs. |
| `generate_answer(question, results)` | Request structured Gemini output and validate reported citations. |
| `ask(question)` | Combine retrieval and generation into one result. |
| `format_answer(result)` | Format the question, generated answer, and filenames for display. |
| `summarize_text(text)` | Summarize supplied text using Gemini. |
| `generate_quiz(results)` | Generate and validate two four-option questions from retrieved context. |
| `start_bot()` / `stop_bot()` | Manage Telegram polling within the notebook event loop. |

### Data Contracts

A retrieved result contains:

```json
{
  "source": "data_warehousing.txt",
  "chunk_id": 1,
  "content": "Retrieved document passage...",
  "score": 0.75,
  "reference_id": "R1"
}
```

This example score is illustrative. Each search assigns reference IDs to its own ranked results.

The `ask()` result contains `question`, `retrieved`, `answer`, and `references`. The `references` list contains unique source filenames mapped from the model's validated reference IDs.

### Persistence and Integrity

`manifest.json` acts as the completion marker for ingestion. It records the embedding model, chunk settings, vector dimensions, chunk count, and SHA-256 hashes of source files and saved database files. Loading detects inconsistent settings, modified source files that remain present, and mismatched saved artifacts.

Rebuilds are explicit. The same embedding model and normalization are used for both stored chunks and incoming questions. JSON metadata avoids loading executable pickle content.

### Bot Execution and State

The Telegram application uses asynchronous handlers. Blocking retrieval and Gemini calls run through `asyncio.to_thread`. Quiz answers and recent results are stored in `context.user_data`; quiz grading is performed locally. Long messages are split into segments before sending.

### Credentials and Data Handling

Keep Gemini and Telegram credentials in Colab Secrets or environment variables. Do not commit keys, tokens, private documents, generated databases containing private text, or notebook outputs that expose sensitive information. Questions and retrieved passages used for generation are sent to Gemini. User messages and bot replies pass through Telegram.

## Git and Version Management

The following workflow is recommended for the repository. These are practices to apply when publishing and maintaining the project, not a claim that repository history has already been created.

### Suggested Repository Structure

```text
SmartAssistant.ipynb
README.md
.gitignore
docs/
  # Optional extended design notes and evaluation records
```

The technical documentation is included in this README. Add longer evaluation reports under `docs/` as the project grows.

### Suggested .gitignore

```gitignore
.env
.env.*
!.env.example
.venv/
__pycache__/
.ipynb_checkpoints/
SmartAssistant_RAG/
*.index
```

Add any additional directories containing private data to `.gitignore`. Review notebook outputs before every commit.

### Initial Commit

From the project directory:

```bash
git init
git add README.md SmartAssistant.ipynb .gitignore
git commit -m "feat: add foundational RAG pipeline and Telegram interface"
git branch -M main
```

Create `.gitignore` before running these commands. Connect the repository to your own GitHub remote when ready.

### Ongoing Changes

- Keep `main` stable and create descriptive branches such as `fix/reference-validation` or `feat/quiz-feedback`.
- Use small, focused commits with messages explaining the change.
- Review changes with `git diff` and check staged files before committing.
- Use pull requests to describe behavior changes and the validation performed.
- Update documentation when configuration or behavior changes.
- Tag reviewed releases using semantic versions, such as `v1.0.0`. Do not label a release as fully tested until live integration checks have passed.

## Community Participation

Explore the [SDAIA Academy GitHub account](https://github.com/SDAIAAcademy) and Saudi open-source projects. Support relevant, high-quality work with stars, follows, constructive issues, forks, or pull requests. Follow each project's contribution guidelines and license before contributing or reusing code.

## License

A license has not yet been selected. Before encouraging reuse or accepting open-source contributions, add a `LICENSE` file that reflects the project owner's intended permissions.
