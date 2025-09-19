
## Summary
Streamlit app that lets users upload a CSV, chat with an AI analyst about the data, and get answers with optional executable Python code and visualizations. It builds a context-aware system prompt using either a compact data summary (for large datasets) or the full dataset, calls OpenAI Chat Completions (model `gpt-4.1`), renders the reply, executes any returned Python code blocks against `df` with `matplotlib`/`seaborn`, captures and displays warnings, and persists chat plus figures. It also provides dataset preview/metrics and can export the entire analysis conversation as a downloadable HTML report.

___
## Example Usage
```bash
# 1) Put your OpenAI key in .streamlit/secrets.toml
# [general]
# OPENAI_API_KEY = "sk-..."

# 2) Save the script as app.py and run:
streamlit run app.py
```

```csv
# Example CSV to upload (sales.csv)
date,region,category,units,price
2024-01-01,North,Widgets,10,19.99
2024-01-02,South,Gadgets,7,24.50
2024-01-03,North,Widgets,12,18.75
2024-01-04,West,Gadgets,5,22.00
```

- In the sidebar, upload sales.csv.
- Ask in the chat: "Show me a correlation matrix for numeric columns and highlight the strongest relationships."
- Expected output:
  - Assistant text summarizing correlations.
  - A heatmap figure rendered below the reply.
  - Any data-validation notes in info boxes (e.g., missing values).
  - The conversation is recorded; click "Generate Report" to download an HTML report of Q&A and metadata.

___
## Code Analysis
### Inputs
- A CSV file uploaded via the sidebar (`st.file_uploader`), parsed into `st.session_state.df`.
- User chat messages entered via `st.chat_input`.
- OpenAI API key from `st.secrets['OPENAI_API_KEY']`.
- Optional: previously generated conversation history in `st.session_state.messages`.
### Flow
1. Configure the Streamlit page, initialize OpenAI client, and set up session state (`messages`, `df`, `data_summary`).
2. When a CSV is uploaded, read into a pandas DataFrame, compute a compact summary (shape, columns, dtypes, sample, stats), and show previews/metrics.
3. If messages exist, enable exporting a styled HTML report via `export_conversation()` and `download_button`.
4. On user question, build a system prompt with either summarized context (>100 rows) or full table (<=100 rows), include recent history (truncated), and call `client.chat.completions.create`.
5. Render the assistant reply; if it contains fenced Python code, execute it with a prepared `matplotlib` figure, capture warnings (`warnings.catch_warnings` + `simplefilter('always')`), display plots and notes, persist message and any figure; handle and explain common execution errors.
### Outputs
- Assistant responses (natural language analysis) displayed in the chat.
- Executed visualization(s) with `matplotlib`/`seaborn` shown inline when code is returned.
- Data previews and summary metrics (rows, columns, memory, missing values).
- Informational messages for captured warnings and friendly error hints on failed code execution.
- Downloadable HTML report of the dataset info and full conversation.

