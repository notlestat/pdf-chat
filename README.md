# PDF Chat

A Streamlit app for asking questions about one PDF through the Anthropic API. It streams answers and displays page ranges and quoted passages when the model returns document citations.

The app uploads the PDF to Anthropic's Files API, attaches its file reference to the first user turn, and includes that turn in later conversation history. It requests prompt caching for the document prefix. There is no vector database or retrieval index.

## Run locally

Use Python 3.10+ in an isolated environment.

```sh
git clone https://github.com/notlestat/pdf-chat.git
cd pdf-chat
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cp .env.example .env
```

On Windows, activate with `.venv\Scripts\Activate.ps1` in PowerShell and use `Copy-Item .env.example .env`.

Set `ANTHROPIC_API_KEY` in `.env`, then run:

```sh
python -m streamlit run app.py
```

Open the local address printed by Streamlit. Upload a PDF and ask a question. The app requires network access and API access to the configured model. PDFs leave your machine when uploaded.

To try the interface with the bundled fixture, use [sample/sample_contract.pdf](sample/sample_contract.pdf). Uploading it still requires API access and can incur API charges.

## Configuration and behavior

| Setting | Current implementation |
|---|---|
| API key | `ANTHROPIC_API_KEY`, loaded from `.env` or the environment |
| Model | `claude-opus-5`, hard-coded as `MODEL` in `pdf_chat/client.py` |
| File API beta | `files-api-2025-04-14` in `pdf_chat/client.py` |
| App page cap | 600 pages in `pdf_chat/documents.py` |
| Large-document notice | 150 pages or more |
| Reasoning effort | Low, medium or high in the sidebar |
| Output token limit | 16,000 in `pdf_chat/chat.py` |
| Cost display | Returned usage multiplied by local price constants in `pdf_chat/chat.py` |

The page cap is an application check, not a guarantee that the provider accepts every PDF below it. Model availability, SDK compatibility, document support and account limits need verification against your API account before use.

The checked-in USD estimates per million tokens are input `5.00`, output `25.00`, cache read `0.50`, and cache write `6.25`. The app does not fetch current provider prices. Verify and update them when changing models or provider rates. Cache hits depend on the provider; the UI reports returned cache-read usage.

## Project files

| File | Purpose |
|---|---|
| [app.py](app.py) | PDF upload, chat, sources, effort control and cost display |
| [pdf_chat/client.py](pdf_chat/client.py) | TLS trust setup, API key loading and client configuration |
| [pdf_chat/documents.py](pdf_chat/documents.py) | PDF validation, page count, fingerprint and file upload |
| [pdf_chat/chat.py](pdf_chat/chat.py) | Conversation history, streaming, citations and usage estimates |
| [requirements.txt](requirements.txt) | Dependency ranges |
| [.env.example](.env.example) | API key placeholder |

## Limits and verification

The prompt asks the model to use the supplied document and state when an answer is missing. That instruction does not guarantee correctness or a citation for every claim. Check the quoted passages against the source PDF before relying on an answer.

Conversation state is held in the Streamlit session. Changing the document resets the conversation. The application reuses an uploaded file within that session but does not implement remote file deletion. There is no authentication layer or automated test suite in the repository, and dependency versions are ranges rather than a lockfile.

A local syntax check is available without making API calls:

```sh
python -m compileall -q app.py pdf_chat
```

Syntax checks do not verify uploads, model access, streaming, citations, or actual costs. To check those behaviors, use your account and a PDF you are permitted to upload. Compare an initial question and a follow-up with the source pages and returned usage.

## Troubleshooting

For a missing-key error, confirm `.env` contains a real key and run from the repository root. For model or request-parameter errors, compare the configured model and beta SDK calls with the provider's current documentation.

`pdf_chat/client.py` calls `truststore.inject_into_ssl()` before constructing the API client to use the operating system's certificate trust store. If HTTPS requests fail, inspect the certificate chain and local trust configuration. Certificate errors can have several causes; do not disable TLS verification.
