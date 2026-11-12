# qasim-1# AI Learning Roadmap Generator

A small Streamlit app that creates a personalised, phase-by-phase learning roadmap.
You enter **what you want to learn**, **your skill level** and **how much time you have**,
and a Gemini Flash model builds the plan.

## Features

- Inputs: field, level (Beginner / Intermediate / Advanced), time to learn (weeks or months) and hours per week
- Structured output: phases with goal, topics, resources, a hands-on project and a milestone
- Week and hour totals are calculated in code, so they always add up to the time you chose
- Rejects nonsense input (for example random characters) with a clear message
- Caches identical requests for one hour to save API quota
- Download any roadmap as a Markdown file
- Optional "bring your own API key" box in the sidebar

## Tech stack

- Python 3.10 or newer
- [Streamlit](https://streamlit.io) for the UI
- [Google Gen AI SDK](https://pypi.org/project/google-genai/) (`google-genai`) with a Gemini Flash model
- [Pydantic](https://docs.pydantic.dev) to define and validate the JSON the model must return

## Project structure

```
.
├── app.py             # the whole application
├── requirements.txt   # Python dependencies
└── README.md
```

## Run it locally

### 1. Get a Gemini API key

Create a free key in [Google AI Studio](https://aistudio.google.com/apikey).

### 2. Install

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

python -m venv .venv

# Windows (PowerShell / CMD)
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

### 3. Add your key

Create the file `.streamlit/secrets.toml` (create the `.streamlit` folder first):

```toml
GEMINI_API_KEY = "your-key-here"
```

Alternatively set an environment variable named `GEMINI_API_KEY`.

> **Never commit your key.** Make sure `.streamlit/secrets.toml` is listed in `.gitignore`.

### 4. Start the app

```bash
streamlit run app.py
```

Streamlit opens http://localhost:8501 in your browser.

## Configuration

| Setting          | Where to set it                            | Default            | Purpose                                  |
| ---------------- | ------------------------------------------ | ------------------ | ---------------------------------------- |
| `GEMINI_API_KEY` | secrets.toml, Streamlit Cloud secrets, env | none (required)    | Your Gemini API key                      |
| `GEMINI_MODEL`   | secrets.toml, Streamlit Cloud secrets, env | `gemini-3.8-flash` | Which Gemini model to call               |

Google renames and retires models over time. If you see a "model not found" message,
look up the current Flash model ID in the
[Gemini models list](https://ai.google.dev/gemini-api/docs/models) and set `GEMINI_MODEL`
to it. No code change is needed.

## How it works

1. `validate_field`, `to_total_weeks` and `validate_hours` clean and check what you typed.
2. `build_user_prompt` turns your inputs into a precise request. All numbers are computed in Python.
3. `generate_roadmap` calls Gemini in JSON mode with a Pydantic schema, so the answer always has the same shape.
4. `distribute_weeks` rescales the model's week counts so they add up exactly to your time budget
   (language models are unreliable at arithmetic, so we do the maths ourselves).
5. The result is stored in `st.session_state` and drawn by `render_roadmap`.

## Deploy on Streamlit Community Cloud

1. Push this project to a **public** GitHub repository (or private, if your account allows it).
2. Go to [share.streamlit.io](https://share.streamlit.io) and sign in with GitHub.
3. Click **Create app**, choose your repository, the `main` branch and `app.py` as the main file.
4. Open **Advanced settings** and paste your secret into the **Secrets** box:

   ```toml
   GEMINI_API_KEY = "your-key-here"
   ```

5. Click **Deploy**. After a few minutes you get a public URL.

To change a secret later: open your app's **Settings -> Secrets**.

## Troubleshooting

| Problem                                | Fix                                                                                   |
| -------------------------------------- | ------------------------------------------------------------------------------------- |
| "No Gemini API key found"              | Add `GEMINI_API_KEY` to secrets, or paste a key in the sidebar                        |
| "Gemini rejected the API key"          | Re-copy the key from AI Studio; make sure there are no spaces or quotes inside it     |
| "model name was not found"             | Set `GEMINI_MODEL` to a current model ID                                              |
| "rate limit or free quota reached"     | Wait a minute; the free tier has per-minute and per-day limits                        |
| `ModuleNotFoundError` on Streamlit Cloud | Check that `requirements.txt` is in the repo root and spelled correctly             |

## Limitations

- Roadmaps are AI-generated guidance, not a guaranteed curriculum. Check resource names before relying on them.
- The app does not generate links on purpose (models tend to invent URLs).
- Maximum time frame is 104 weeks (about two years).

## License

Choose a license for your project (for example MIT) and add a `LICENSE` file.
