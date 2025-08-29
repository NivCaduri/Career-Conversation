# Career Conversation – Niv Caduri

An AI-powered career Q\&A chatbot that answers questions about **Niv Caduri’s** background, skills, and experience. Built with **Python**, **Gradio**, and the **OpenAI API**, and designed to run locally or on **Hugging Face Spaces**. The app can also record user interest (email) and log unknown questions via **Pushover** notifications.&#x20;

## 🚀 Live Demo
👉 [Try it here on Hugging Face Spaces](https://huggingface.co/spaces/nivcaduri/career_conversation)

## Features

* **Persona-accurate responses**: System prompt composes from a summary file and full resume text (PDF) to answer as “Niv Caduri.”
* **Tool calls**:

  * `record_user_details(email, name?, notes?)` → sends a Pushover notification when a user shares contact info.
  * `record_unknown_question(question)` → logs questions that couldn’t be answered.
* **Gradio chat UI**: Simple chat interface exposed via `gr.ChatInterface`.
* **Hugging Face–friendly**: Runs as a single script and launches a Space with `gradio`.

## Tech Stack

* Python, Gradio, OpenAI API, python-dotenv, requests, PyPDF2 (via `pypdf`)

## Project Structure

```
.
├─ app.py
└─ me/
   ├─ resume.pdf      # parsed at startup
   └─ summary.txt     # injected into system prompt
```

`app.py` loads `me/resume.pdf` (concatenates all page text) and `me/summary.txt` to craft the AI’s system prompt. If either is missing, startup will fail or produce incomplete context.

## Getting Started (Local)

### 1) Prerequisites

* Python 3.10+ recommended
* An **OpenAI API key**
* (Optional) **Pushover** app token & user key if you want notifications

### 2) Install dependencies

Create a virtualenv (recommended), then:

```bash
pip install openai python-dotenv requests pypdf gradio
```

### 3) Environment variables

Create a `.env` file in the project root:

```dotenv
# OpenAI
OPENAI_API_KEY=sk-...

# Optional: Pushover (for notifications)
PUSHOVER_TOKEN=...
PUSHOVER_USER=...
```

> The app imports `dotenv` and loads env vars at startup (`load_dotenv(override=True)`).

### 4) Provide profile content

Add your files under `me/`:

```
me/resume.pdf      # your resume PDF
me/summary.txt     # short text summary used in the system prompt
```

> `app.py` reads *all* pages of `me/resume.pdf` and the entire `me/summary.txt`.

### 5) Run

```bash
python app.py
```

Gradio will start a local web UI.

## How It Works (High-Level)

* On startup, `Me()`:

  * Initializes the OpenAI client and loads persona content from `me/resume.pdf` and `me/summary.txt`.
  * Prepares a system prompt instructing the model to act as **Niv Caduri** and to use tools when appropriate (record emails, log unknown questions).
* On each user message:

  * The app calls `openai.chat.completions.create` with the system/user/history messages and the tool schemas.
  * If the model returns tool calls, the app executes them (`requests.post` to Pushover) and sends the tool results back to the model before returning the final answer.
* Gradio’s `ChatInterface` wires this up to a browser chat.

## Deploying to Hugging Face Spaces

1. Create a new **Space** (Python / Gradio).
2. Push these files:

   * `app.py`
   * `me/resume.pdf`
   * `me/summary.txt`
   * `requirements.txt` (see below)
3. Add the following **Secrets** (Settings → Repository secrets):

   * `OPENAI_API_KEY`
   * (Optional) `PUSHOVER_TOKEN`, `PUSHOVER_USER`
4. Set the **Space SDK** to Gradio (default for Python/Gradio templates).
5. Space will build and run automatically. `gr.ChatInterface(...).launch()` works in Spaces by default.

### Sample `requirements.txt`

```
openai
python-dotenv
requests
pypdf
gradio
```

## Security Notes

* Never commit your API keys or Pushover credentials. Use `.env` locally and **HF Secrets** on Spaces.
* The app logs tool usage (via Pushover). Keep messages generic if privacy is a concern.

## Customization

* **Persona content**: Edit `me/summary.txt` and `me/resume.pdf`.
* **System prompt**: Modify `system_prompt()` to tweak tone or behavior.
* **Tools**: Extend `tools` with new functions and add corresponding JSON schemas for model tool-calling.

## Troubleshooting

* **Gradio not launching**: Confirm dependencies installed; ensure `python -c "import gradio"` works.
* **OpenAI errors**: Check `OPENAI_API_KEY` exists and is valid.
* **Missing context**: Ensure `me/resume.pdf` and `me/summary.txt` exist and are readable.
* **Pushover errors**: Remove or unset `PUSHOVER_*` if you don’t need notifications, or verify both token and user keys.

## License

Personal/portfolio project. Adapt as needed.